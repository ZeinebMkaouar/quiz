<template>
  <footer>
    <nav class="nav-container" role="navigation" aria-label="Navigation buttons">
      <div class="nav-steps">
        <button
          v-for="(question, index) in questions"
          :key="index"
          type="button"
          :disabled="chosenAnswers.length < index"
          :class="['button', { 'is-active': currentQuestionIndex == index }]"
          @click="selectQuestion(index)"
        >
          {{ index + 1 }}
        </button>
      </div>
      <div class="nav-buttons">
        <button
          class="button"
          type="button"
          @click="prevQuestion();"
          :disabled="currentQuestionIndex < 1"
        >Back</button>
        <button
          class="button button--next"
          type="button"
          @click="nextQuestion();"
          :disabled="chosenAnswers[currentQuestionIndex] == null || currentQuestionIndex >= questions.length"
        >{{ currentQuestionIndex == questions.length - 1 ? 'Submit' : 'Next' }}</button>
      </div>
    </nav>
  </footer>
</template>

<script>
export default {
  name: 'FooterNav',
  props: {
    questions: Array,
    chosenAnswers: Array,
    currentQuestionIndex: Number,
  },
  methods: {
    prevQuestion() {
      if (this.questions.length > 0 && this.currentQuestionIndex > 0) {
        this.$emit('update-question-index', this.currentQuestionIndex - 1);
      }
    },
    nextQuestion() {
      if (this.currentQuestionIndex == this.questions.length - 1) {
        this.$emit('submit-quiz');
      } else if (this.currentQuestionIndex < this.questions.length) {
        this.$emit('update-question-index', this.currentQuestionIndex + 1);
      }
    },
    selectQuestion(index) {
      if (this.chosenAnswers.length >= index) {
        this.$emit('update-question-index', index);
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.button  {
 
  width: 3rem; 
  height: 2rem;

  border: none;  
  &--next {
    color: #fff;
    background-color: $btn-bg--active;
    &:hover {
      background-color: $btn-hover;
    }
  }
}

.nav-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 3rem;
  
  .nav-steps {
    display: flex;
    gap: 10px;

    .button {
      width: 2rem;
      height: 2rem;
      min-width: auto;
      padding: 0;
    }

    .is-active {
      color: #fff;
      background-color: $btn-bg--active;
    }
  }
  
  .nav-buttons {
    display: flex;
    gap: 10px;
  }
}
</style>
