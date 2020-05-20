<template>
  <div class="survey-answers">
    <div class="input-group add--display-flex">
      <div class="input-textfield">
        <input type="text" v-model="searchword">
        <v-icon v-if="searchword === ''" name="search" class="search-icon" />
        <v-icon v-else name="times-circle" class="delete-icon" @click="clearSearch()" />
      </div>
    </div>
    <div class="answers-container">
      <!-- Answer List -->
      <div class="answers-container__answer-list">
        <table class="minimalistic-table">
          <tr>
            <th @click="sortBy('name')">
              <div class="flex-wrapper">name
                <span class="sort-container">
                  <v-icon class="sort-indicator" :class="[currentSortCategory === 'name' && currentSortDirection === 'ascending' ? 'sort-active' : '']" name="caret-up" scale="0.8"/>
                  <v-icon class="sort-indicator" :class="[currentSortCategory === 'name' && currentSortDirection === 'descending' ? 'sort-active' : '']" name="caret-down" scale="0.8"/>
                </span>
              </div>
            </th>
            <th @click="sortBy('timestamp')">
              <div class="flex-wrapper">date
                <span class="sort-container">
                  <v-icon class="sort-indicator" :class="[currentSortCategory === 'timestamp' && currentSortDirection === 'ascending' ? 'sort-active' : '']" name="caret-up" scale="0.8"/>
                  <v-icon class="sort-indicator" :class="[currentSortCategory === 'timestamp' && currentSortDirection === 'descending' ? 'sort-active' : '']" name="caret-down" scale="0.8"/>
                </span>
              </div>
            </th>
            <th @click="sortBy('score')">
              <div class="flex-wrapper">score
                <span class="sort-container">
                  <v-icon class="sort-indicator" :class="[currentSortCategory === 'score' && currentSortDirection === 'ascending' ? 'sort-active' : '']" name="caret-up" scale="0.8"/>
                  <v-icon class="sort-indicator" :class="[currentSortCategory === 'score' && currentSortDirection === 'descending' ? 'sort-active' : '']" name="caret-down" scale="0.8"/>
                </span>
              </div>
            </th>
          </tr>
          <tr v-for="(answer) in sortedAnswers" :key="answer.id" @click="selectAnswer(answer.id)" :class="[answerSelected != null && answer.id === sortedAnswers[answerSelected].id ? 'answer-selected' : '']">
            <td v-if="answer.name">{{ answer.name }}</td>
            <td v-else>Name Unavailable</td>
            <td>
              <span v-if="answer.date">{{ answer.date }}</span>
              <span v-else>Date Unavailable</span>
            </td>
            <td><span class="score-container">
              {{ answer.score }}
              <!-- smile-emoji -->
              <v-icon v-if="answer.score >= 9" class="score-icon fa-smile" name="regular/smile" scale="1.5"/>
              <!-- frown-emoji -->
              <v-icon v-else-if="answer.score <= 6" class="score-icon fa-frown" name="regular/frown" scale="1.5"/>
              <!-- meh-emoji -->
              <v-icon v-else class="score-icon fa-meh" name="regular/meh" scale="1.5"/>
              </span>
            </td>
          </tr>
        </table>
        <div class="button-container" v-if="loadCount <= answers.length">
          <button class="button-neutral" @click="loadMoreAnswers()">Load More</button>
        </div>
      </div>

      <!-- Answer Container -->
      <div class="answers-container__answer-container" v-if="answerSelected != null">
        <div class="answer-inner">
          <!-- Answer Container / Header -->
          <div class="answer-inner__header">
            <h2 v-if="answers[answerSelected].name" class="h2">{{ answers[answerSelected].name }}</h2>
            <h2 v-else class="h2">Name Unavailable</h2>
            <div class="close-icon" @click="unselectAnswer()">
              <v-icon name="times"/>
            </div>
          </div>
          <!-- Answer Container / Main -->
          <div class="answer-inner__main">
            <div class="answer-inner__main__col-2">
              <div class="answer-inner__detail-container">
                <span class="answer-inner__detail-container__detail-header">responded</span>
                <p v-if="answers[answerSelected].date">{{answers[answerSelected].date}}</p>
                <p v-else>Date Unavailable</p>
              </div>
              <div class="add--display-flex">
                <div class="score-container">
                  <!-- smile-emoji -->
                  <v-icon v-if="answers[answerSelected].score >= 9" class="score-icon fa-smile" name="regular/smile" scale="2.5"/>
                  <!-- frown-emoji -->
                  <v-icon v-else-if="answers[answerSelected].score <= 6" class="score-icon fa-frown" name="regular/frown" scale="2.5"/>
                  <!-- meh-emoji -->
                  <v-icon v-else class="score-icon fa-meh" name="regular/meh" scale="2.5"/>
                </div>
                <div class="answer-inner__detail-container">
                  <span class="answer-inner__detail-container__detail-header">score</span>
                  <p>{{answers[answerSelected].score}}</p>
                </div>
              </div>
            </div>
            <div class="answer-inner__detail-container">
              <span class="answer-inner__detail-container__detail-header">liked</span>
              <p v-if="answers[answerSelected].likes.length != 0">{{ convertLikesToString(answerSelected) }}</p>
              <p v-else>No Likes Selected</p>
            </div>
            <div class="answer-inner__detail-container">
              <span class="answer-inner__detail-container__detail-header">disliked</span>
              <p v-if="answers[answerSelected].dislikes.length != 0">{{ convertDislikesToString(answerSelected) }}</p>
              <p v-else>No Dislikes Selected</p>
            </div>
            <div class="answer-inner__detail-container">
              <span class="answer-inner__detail-container__detail-header">email</span>
              <p v-if="answers[answerSelected].contact">{{ answers[answerSelected].email }}</p>
              <p v-else-if="!answers[answerSelected].contact">May Not Contact</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import FetchingLoader from '@/components/GlobalComponents/FetchingLoader';
import { firestore } from "@/firebase";

export default {
  components: {
    // FetchingLoader,
  },
  data() {
    return {
      answers: [],
      answerSelected: null,
      currentSortCategory: "timestamp",
      currentSortDirection: "descending",
      searchword: "",
      loadCount: 10,
    }
  },
  computed: {
    monthsInString() {
      return ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"];
    },
    filteredAnswers() {
      let filteredAnswers = this.answers;

      if (this.searchword) {
        filteredAnswers = this.answers.filter(answer =>
          answer.name.toLowerCase().indexOf(this.searchword.toLowerCase()) > -1
        );
      }
      
      return filteredAnswers;
    },
    sortedAnswers() {
      // Sort answers array, slice to copy it without reference ('...' no work in computed)
      return this.filteredAnswers.slice(0).sort((a, b) => {
        const category = this.currentSortCategory;
        let valueA = a[category];
        let valueB = b[category];

        // Error prevention: Make sure values are never undefined
        // If undefined, always last
        if (!valueA && category === "name") {
          valueA = "zzz";
        } else if (!valueA) {
          valueA = -1;
        }

        if (!valueB && category === "name") {
          valueB = "zzz";
        } else if (!valueB) {
          valueB = -1;
        }

        const direction = this.currentSortDirection;
        let modifier = 1;

        // Check if descending. If so, invert everything with modifier
        if(direction === 'descending') {
          modifier = -1;
        }

        // toLowerCase() if string
        if (category === "name") {
          if (valueA.toLowerCase() < valueB.toLowerCase()) {
            return -1 * modifier;
          }
          if (valueA.toLowerCase() > valueB.toLowerCase()) {
            return 1 * modifier;
          }
        } else {
          if (valueA < valueB) {
            return -1 * modifier;
          }
          if (valueA > valueB) {
            return 1 * modifier;
          }
        }
        return 0;

      });
    }
  },
  created() {
    this.getAnswers()
  },
  methods: {
    getAnswers() {
      let answers = []

      firestore.collection("survey")
      .doc(this.$hotelId)
      .collection("answers")
      .orderBy("timestamp", "desc")
      .limit(this.loadCount)
      .get()
      .then(snapshot => {        
        snapshot.forEach(doc => {
          let answer = doc.data();
          answer.id = doc.id;
          if (answer.timestamp) {
            answer.date = `${new Date(answer.timestamp).getDate()} 
            ${this.monthsInString[new Date(answer.timestamp).getMonth()]} 
            ${new Date(answer.timestamp).getFullYear()}`
          }

          answers.push(answer)
        })
        this.answers = answers;
      })
      .catch(error => {
        console.log(error);
      })
    },
    loadMoreAnswers() {
      this.loadCount += 10;
      this.getAnswers();
    },
    clearSearch() {
      this.searchword = "";
    },
    selectAnswer(id) {
      console.log(id);
      
      const index = this.answers.findIndex(answer => answer.id === id)
      console.log(index);
      this.answerSelected = index;
    },
    unselectAnswer() {
      this.answerSelected = null;
    },
    convertLikesToString(index) {
      let selectedOptions = "";
      const answer = this.answers[index];

      answer.likes.forEach((like, index) => {
        if (index === answer.likes.length - 1) {
          selectedOptions += like;
        } else {
          selectedOptions += `${like}, `;
        }
      });

      if (answer.likesOther) {
        selectedOptions += ` (${answer.likesOther})`
      }

      return selectedOptions;
    },
    convertDislikesToString(index) {
      let selectedOptions = "";
      const answer = this.answers[index];

      answer.dislikes.forEach((dislike, index) => {
        if (index === answer.dislikes.length - 1) {
          selectedOptions += dislike;
        } else {
          selectedOptions += `${dislike}, `;
        }
      });

      if (answer.dislikesOther) {
        selectedOptions += ` (${answer.dislikesOther})`
      }

      return selectedOptions;
    },
    sortBy(category) {
      // If already category clicked, reverse direction
      if (category === this.currentSortCategory) {
        this.currentSortDirection = this.currentSortDirection === "ascending" ? "descending" : "ascending"

      } else if (category === "name") {
        // If switched to name, start as ascending (A-Z)
        this.currentSortCategory = category;
        this.currentSortDirection = "ascending";

      } else if (category === "timestamp") {
        // If switched to date, start as descending (Newest)
        this.currentSortCategory = category;
        this.currentSortDirection = "descending"
        
      } else if (category === "score") {
        // If switched to date, start as ascending (0-10)
        this.currentSortCategory = category;
        this.currentSortDirection = "ascending"
      }

      // Actual sorting happens in computed

    }
  }
}
</script>

<style scoped>

.input-textfield input {
  min-height: 32px;
  font-size: 16px;
}

.search-icon,
.delete-icon {
  position: absolute;
  top: 8px;
  right: 8px;
}

.search-icon {
  pointer-events: none;
}

.delete-icon {
  cursor: pointer;
}

.fa-smile {
  color: var(--primary);
}

.fa-meh {
  color: var(--grey);
}

.fa-frown {
  color: var(--alert);
}

.minimalistic-table tr:hover {
  background-color: var(--grey-light);
  cursor: pointer;
}

.minimalistic-table tr:first-child:hover {
  background-color: var(--light);
  cursor: default;
}

.minimalistic-table tr th .flex-wrapper {
  display: flex;
  justify-content: flex-end;
}

.minimalistic-table tr th:first-child .flex-wrapper {
  justify-content: flex-start;
}

.minimalistic-table tr th:hover {
  background-color: var(--grey-light);
}

.minimalistic-table .sort-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 4px;
}

.minimalistic-table .sort-container .sort-indicator:first-child {
  /* Bad way of reducing the distance between them, but it works */
  margin-bottom: -7px;
}

.minimalistic-table .sort-container .sort-active {
  color: var(--primary)
}

.minimalistic-table tr td {
  white-space: nowrap;
}

.minimalistic-table .score-container {
  display: flex;
  justify-content: flex-end;
}

.minimalistic-table .score-container .score-icon {
  margin-left: 8px;
}

.minimalistic-table .answer-selected {
  background: var(--grey-light);
  font-weight: var(--weight-medium);
}

.answers-container {
  display: flex;
}

.answers-container__answer-list {
  width: 100%;
}

.button-container {
  justify-content: center;
}

.answers-container__answer-container {
  width: 384px;
  padding-left: 32px;
  flex-flow: 0;
  flex-shrink: 0;
}

.answer-inner {
  padding: 16px;
  position: sticky;
  top: 0;
}

.answer-inner__header {
  display: flex;
  justify-content: space-between;
}

.answer-inner__header h2 {
  margin-top: 0;
}

.answer-inner__header .close-icon {
  width: 16px;
  height: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 21px;
  color: var(--grey);
  cursor: pointer;
}

.answer-inner__detail-container {
  line-height: 1.25;
}

.answer-inner__detail-container__detail-header {
  color: var(--grey);
  font-size: 0.75rem;
}

.answer-inner__main__col-2 {
  display: flex;
}

.answer-inner__main__col-2 .answer-inner__detail-container {
  width: 50%;
}

.answers-container__answer-container .score-container svg {
  width: 40px;
  height: 40px;
  position: relative;
  margin-right: 8px;
}

</style>