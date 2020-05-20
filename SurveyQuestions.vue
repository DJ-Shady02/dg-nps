<template>
  <section class="survey-box">
      <!-- :class="{
        'question-type__main': question.type === 'main',
        'question-type__options': question.type === 'options',
        'question-type__contact': question.type === 'contact',
        'question-type__confirmation': question.type === 'confirmation',
      }" -->
    <div v-if="questionIndex >= Object.keys(surveyQuestions).length">
      <div class="survey-container">
        <p>Your answer was sent 👍</p>
        <p>A big thanks from us at QA Hotel</p>
      </div>
    </div>
    <div v-else>
      <div
        class="survey-container"
        v-for="(question, index) in surveyQuestions"
        :key="index"
        v-show="questionIndex == index"
      >
        <!-- Main Question -->
        <main-question v-if="question.type === 'main'" :question="question" @passValue="storeValue(...arguments)"/>

        <!-- Options Question -->
        <options-question v-else-if="question.type === 'options'" :question="question" :options="surveySettings.options" @passValue="storeValue(...arguments)"/>

        <!-- Contact Question -->
        <contact-question v-else-if="question.type === 'contact'" :question="question" @passValue="storeValue(...arguments)"/>

      </div>
      <div class="button-container" v-if="questionIndex !== 0">
        <!-- Back button -->
        <button v-if="questionIndex < Object.keys(surveyQuestions).length" class="button-neutral" @click="decreaseIndex()">BACK</button>

        <!-- Next button -->
        <button v-if="questionIndex === Object.keys(surveyQuestions).length - 1" class="button-positive" @click="sendToFirestore()">SEND</button>
        <button v-else-if="questionIndex < Object.keys(surveyQuestions).length" class="button-positive" @click="increaseIndex()">NEXT</button>
      </div>
    </div>
  </section>
</template>

<script>
import { db, firestore } from "@/firebase";
import cookieMixin from "@/mixins/cookieMixin";
import MainQuestion from "../components/SurveyTypes/MainQuestion";
import OptionsQuestion from "../components/SurveyTypes/OptionsQuestion";
import ContactQuestion from "../components/SurveyTypes/ContactQuestion";

export default {
  // props: {
  //   customFonts: Object,
  //   secondLang: Boolean,
  //   guestPhase: String,
  //   cookieExists: [String]
  // },
  components: {
    MainQuestion,
    OptionsQuestion,
    ContactQuestion
  },
  mixins: [cookieMixin],
  data() {
    return {
      // questionsFetched: false,
      // activeGuestTemp: "",
      // surveyQuestions: {
      //   0: {
      //     text: "Hvor sandsynligt er det, at du vil anbefale QA Hotel til en ven?",
      //     type: "main"
      //   },
      //   1: {
      //     text: "Hvad gjorde vi godt?",
      //     type: "options",
      //     favour: "like",
      //     options: ["Service", "Mad", "Rengøring", "Værelset", "Booking", "Andet"]
      //   },
      //   2: {
      //     text: "Hvad kan vi gøre bedre?",
      //     type: "options",
      //     favour: "dislike",
      //     options: ["Service", "Mad", "Rengøring", "Værelset", "Booking", "Andet"]
      //   },
      //   3: {
      //     text: "Må vi kontakte dig med opfølgende spørgsmål?",
      //     type: "contact"
      //   }
      // },
      questionIndex: 0,
      answerScore: 0,
      answerLikes: [],
      answerLikesOther: "",
      answerDislikes: [],
      answerDislikesOther: "",
      answerContact: false,
      answerContactEmail: ""
    };
  },
  firebase() {
    return {
      activeGuestInformation: {
        source: db.ref('activeguests/' + this.$route.params.hotel + '/' + this.$route.params.guestid),
        asObject: true
      },
      surveyQuestions: db.ref(`survey/${this.$route.params.hotel}/questions`),
      surveySettings: {
        source: db.ref(`survey/${this.$route.params.hotel}/settings`),
        asObject: true
      }
    };
  },
  mounted() {
    console.log(this.surveySettings);
    console.log(this.surveyQuestions);
    
    
  },
  methods: {
    storeValue(value, secondValue) {
      const question = this.surveyQuestions[this.questionIndex];
      // console.log(value, secondValue);
      
      // console.log("type is " + question.type);
      // console.log("value is " + value);
      if (question.type === "main") {
        this.answerScore = value;
        console.log("answerScore = " + this.answerScore);
        this.increaseIndex();

      } else if (question.type === "options" && question.favour === "like") {
        this.answerLikes = value;
        this.answerLikesOther = secondValue;
        console.log("answerLikes = " + this.answerLikes);
        console.log("answerLikesOther = " + this.answerLikesOther);
        
      } else if (question.type === "options" && question.favour === "dislike") {
        this.answerDislikes = value;
        this.answerDislikesOther = secondValue;
        console.log("answerDislikes = " + this.answerDislikes);
        console.log("answerDislikesOther = " + this.answerDislikesOther);
        
      } else if (question.type === "contact") {
        this.answerContact = value;
        this.answerContactEmail = secondValue;
        console.log("answerContact = " + this.answerContact);
        console.log("answerContactEmail = " + this.answerContactEmail);
      }
    },
    sendToFirestore() {
      // Create a new document with generated ID and save database reference
      const answerRef = firestore.collection('survey')
        .doc(this.$route.params.hotel)
        .collection("answers")
        .doc();

      // Create object to send with static properties
      let answerObject = {
        name: this.getCookie("guestname"),
        score: this.answerScore,
        likes: this.answerLikes,
        dislikes: this.answerDislikes,
        contact: this.answerContact,
        timestamp: Date.now()
      }

      // Add conditional properties
      if (this.answerContact) {
        answerObject.email = this.answerContactEmail;
      }
      if (this.answerLikesOther) {
        answerObject.likesOther = this.answerLikesOther;
      }
      if (this.answerDislikesOther) {
        answerObject.dislikesOther = this.answerDislikesOther;
      }
      if (!this.getCookie("guestname")) {
        answerObject.name = "";
      }

      console.log("About to set object!");
      
      
      // Add answer to the document
      answerRef.set(answerObject)
      .then(() => {
        // Display confirmation and thank you
        this.increaseIndex()
        console.log("Answer was sent to database");
        console.log(answerObject);
        
      })
      .catch((error) => {
        console.log("Something went wrong!", "Answer was not sent to database");
        console.log(error);
        
        
      });
    },
    increaseIndex() {
      this.questionIndex++;
      console.log("Index was increased to: " + this.questionIndex);
    },
    decreaseIndex() {
      this.questionIndex--;
      console.log("Index was decreased to: " + this.questionIndex);
    },
    logValue(value, secondValue) {
      console.log("Logging value from logValue:");
      console.log(value);
      console.log(secondValue);
      
    }
  }
};
</script>

<style scoped>

.survey-box {
  height: calc(100vh - 70px - 15px);
  max-width: 592px; /* 11 buttons + width between them all + 32 px for padding */
  padding: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  font-size: 16px;
  color: #707070;
  margin: 0 auto;
}

.button-container {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}

.button-container button {
  min-width: 120px;
  height: 45px;
  border-radius: 8px;
  border: none;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.25);
  cursor: pointer;
}

.button-container button:last-child {
  margin-left: 16px;
}

.button-neutral {
  background-color: white;
  color: #707070;
}

.button-positive {
  background-color: #0C577E;
  color: white;
}

.survey-container {
  background: white;
  width: 100%;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.25);
  padding: 16px;
}

</style>
