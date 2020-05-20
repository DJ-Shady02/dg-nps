<template>
  <div class="main-question survey-container__question-main">
    <div class="survey-container__question-container">
      {{question.text}}
    </div>
    <div class="survey-container__answer-container">
      <div
        class="survey-container__answer-container__answer"
        v-for="index in 11"
        :key="index"
        @click="$emit('passValue', index - 1)"
      >
        {{ index - 1 }}
      </div>
    </div>
    <div class="survey-container__explanation-container">
      <div>0 - Not likely at all</div>
      <div>10 - Extremely likely</div>
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
      //
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

.survey-container__question-main
.survey-container__answer-container {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;}

.survey-container__question-main
.survey-container__answer-container__answer {
  width: 40px;
  height: 40px;
  border: 1px solid #aaaaaa;
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 4px 8px 4px;
  color: #0C577E;
  font-size: 20px;
}

.survey-container__question-main
.survey-container__explanation-container {
  margin-top: 8px;
  display: flex;
  justify-content: space-between;
}

@media screen and (min-width: 768px) {
  
  .survey-container__question-main
  .survey-container__answer-container__answer:first-child {
    margin-left: 0;
  }

  .survey-container__question-main
  .survey-container__answer-container__answer:last-child {
    margin-right: 0;
  }

}

</style>