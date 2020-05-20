<template>
  <div class="contact-question survey-container__question-contact">
    <div class="survey-container__question-container">
      {{question.text}}
    </div>
    <div class="survey-container__answer-container">
      <label>
        <input type="radio" :value="false" v-model="isContactAllowed" @change="$emit('passValue', isContactAllowed, guestEmail)">
        <div class="survey-container__answer-container__answer">Nej</div>
      </label>
      <label>
        <input type="radio" :value="true" v-model="isContactAllowed" @change="$emit('passValue', isContactAllowed, guestEmail)">
        <div class="survey-container__answer-container__answer">Ja</div>
      </label>
      <div class="survey-container__contact-question-container" v-if="isContactAllowed">
        <div class="survey-container__question-container">
          Hvor kan vi få fat på dig?
        </div>
        <input class="survey-container__answer-container__textfield" v-model="guestEmail" placeholder="email..." @change="$emit('passValue', isContactAllowed, guestEmail)">
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    question: Object
  },
  data() {
    return {
      isContactAllowed: 0,
      guestEmail: ""
    }
  },
  methods: {
    //
  }
};
</script>

<style scoped>

.survey-container__question-container {
  margin-bottom: 16px;
}

.survey-container__answer-container__answer {
  cursor: pointer;
}

.survey-container__explanation-container {
  color: #aaaaaa;
  font-size: 12px;
}

.survey-container__answer-container__textfield {
  outline: none;
  width: 100%;
  border: none;
  border-bottom: 1px solid #707070;
  color: #707070;
}

.survey-container__answer-container__textfield:focus {
  border-bottom-color: #0C577E;
}

.survey-container__answer-container__textfield::placeholder {
  color: #aaaaaa;
}

.survey-container__question-contact
.survey-container__answer-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}

.survey-container__question-contact
.survey-container__answer-container label {
  width: 46.5%;
  margin: 0 4px 8px 4px;
}

.survey-container__question-contact
.survey-container__answer-container label input {
  display: none;
}

.survey-container__question-contact
.survey-container__answer-container__answer {
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 8px;
  border: 1px solid #aaaaaa;
  color: #0C577E;
}

.survey-container__question-contact
.survey-container__answer-container input:checked ~
.survey-container__answer-container__answer {
  background-color: #0C577E;
  color: white;
  border: none;
}

.survey-container__contact-question-container {
  width: 100%;
  margin-top: 8px;
}

.survey-container__contact-question-container
.survey-container__question-container {
  margin-bottom: 8px;
}

</style>