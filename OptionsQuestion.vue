<template>
  <div class="options-question survey-container__question-options">
    <div class="survey-container__question-container">
      {{question.text}}
    </div>
    <div class="survey-container__explanation-container">
      <div>Select any that apply</div>
    </div>
    <div class="survey-container__answer-container">
      <label
        v-for="(option, index) in options"
        :key="index"
      >
        <!-- Other button -->
        <input v-if="index === options.length - 1" type="checkbox" :value="option" v-model="selectedOptions" @change="$emit('passValue', selectedOptions, otherAnswer), isOtherSelected = !isOtherSelected">
        <!-- Normal buttons -->
        <input v-else type="checkbox" :value="option" v-model="selectedOptions" @change="$emit('passValue', selectedOptions, otherAnswer)">
        <div class="survey-container__answer-container__answer">{{option}}</div>
      </label>
      <input class="survey-container__answer-container__textfield" v-if="isOtherSelected" v-model="otherAnswer" placeholder="..." @change="$emit('passValue', selectedOptions, otherAnswer)">
    </div>
  </div>
</template>

<script>
export default {
  props: {
    question: Object,
    options: Array
  },
  data() {
    return {
      isOtherSelected: false,
      selectedOptions: [],
      otherAnswer: ""
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

.survey-container__question-options
.survey-container__answer-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}

.survey-container__question-options
.survey-container__answer-container label {
  width: 46.5%;
  margin: 0 4px 8px 4px;
}

.survey-container__question-options
.survey-container__answer-container label input {
  display: none;
}

.survey-container__question-options
.survey-container__answer-container__answer {
  height: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 8px;
  border: 1px solid #aaaaaa;
  color: #0C577E;
}

.survey-container__question-options
.survey-container__answer-container input:checked ~
.survey-container__answer-container__answer {
  background-color: #0C577E;
  color: white;
  border: none;
}

.survey-container__question-options
.survey-container__explanation-container {
  margin-bottom: 8px;
}

@media screen and (min-width: 768px) {
  
  .survey-container__question-options
  .survey-container__answer-container label {
    width: 31.5%;
  }

}

</style>