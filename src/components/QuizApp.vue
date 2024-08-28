<template>
  <div>
    <!-- Quiz Setup Section -->
    <div class="quiz-setup" v-if="!isStarted">
      <h1>Start your Quiz</h1>

      <h2>Difficulty</h2>
      <div class="difficulty-options">
        <div
          class="difficulty-option"
          :class="{ selected: chosenDifficulty === difficulty.level }"
          v-for="(difficulty, index) in difficulties"
          :key="index"
          @click="setDifficulty(difficulty.level)"
        >
          {{ difficulty.name }}
        </div>
      </div>

      <h2>Category</h2>
      <div class="categories-container">
        <div
          class="setup-option"
          :class="{ 'is-selected': chosenCategory === null }"
          @click="setCategory(null)"
        >
          Any Category
        </div>
        <div
          class="setup-option"
          :class="{ 'is-selected': chosenCategory === category.id }"
          v-for="category in categories"
          :key="category.id"
          @click="setCategory(category.id)"
        >
          {{ category.name }}
        </div>
      </div>

      <h2>Type</h2>
      <div class="types-container">
        <div
          class="type-option"
          :class="{ 'is-selected': chosenType === type.type }"
          v-for="(type, index) in types"
          :key="index"
          @click="setType(type.type)"
        >
          {{ type.name }}
        </div>
      </div>

      <h2>Number Of Questions</h2>
      <div class="types-container">
        <div
          class="setup-option"
          :class="{ 'is-selected': chosenNum === num }"
          v-for="(num, index) in nums"
          :key="index"
          @click="setNum(num)"
        >
          {{ num }}
        </div>
      </div>

      <button class="button button--start" type="button" @click="startQuiz">
        Start
      </button>
    </div>

    <!-- Quiz Section -->
    <div v-if="isStarted">
      <div v-if="questions.length === 0">
        <LoadingApp text="Thinking of questions..." />
      </div>
      <div v-else>
        <div v-if="currentQuestionIndex < questions.length && timer > 0">
          <div class="timer">Time Remaining: {{ timer }} seconds</div>
          <div class="quiz" v-if="questions[currentQuestionIndex]">
            <h2>
              {{ decodeHtml(questions[currentQuestionIndex]?.text || "") }}
            </h2>
            <div class="answers-container">
              <div
                class="answer answer--option"
                v-for="(answer, index) in questions[currentQuestionIndex]
                  ?.answers || []"
                @click="selectAnswer(index)"
                :class="{
                  'is-selected': chosenAnswers[currentQuestionIndex] === index,
                }"
                :key="index"
              >
                {{ decodeHtml(answer?.text || "") }}
              </div>
            </div>
            <FooterNav
              :questions="questions"
              :chosenAnswers="chosenAnswers"
              :currentQuestionIndex="currentQuestionIndex"
              @update-question-index="updateQuestionIndex"
              @submit-quiz="submitQuiz"
            />
          </div>
        </div>
        <div v-if="isSubmitted || timer <= 0" class="quiz-completed">
          <p class="score">{{ calcScore() }} / {{ questions.length }}</p>
          <h2 class="completion-message">{{ completionMessage() }}</h2>
          <div class="quiz-answers">
            <div
              class="quiz-answer"
              v-for="(question, index) in questions"
              :key="index"
            >
              <h3 class="question">{{ decodeHtml(question?.text || "") }}</h3>
              <p class="answer answer--correct">
                Correct answer:
                {{
                  decodeHtml(
                    question?.answers.find((a) => a.correct)?.text || ""
                  )
                }}
              </p>
              <p
                v-if="!answerCorrect(question, chosenAnswers[index])"
                class="answer answer--incorrect"
              >
                Your answer:
                {{
                  decodeHtml(
                    question?.answers[chosenAnswers[index]]?.text || ""
                  )
                }}
              </p>
            </div>
          </div>
          <div class="restart-buttons">
            <button class="button" type="button" @click="reset()">
              Try again
            </button>
            <button class="button" type="button" @click="resetQuestions()">
              Restart with new questions
            </button>
            <button class="button" type="button" @click="resetAll()">
              New setup
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import LoadingApp from "./LoadingApp.vue";
import FooterNav from "./FooterNav.vue";

const difficulties = [
  { level: null, name: "Any Difficulty" },
  { level: "easy", name: "Easy" },
  { level: "medium", name: "Medium" },
  { level: "hard", name: "Hard" },
];

const types = [
  { type: null, name: "Any Type" },
  { type: "multiple", name: "Multiple Choice" },
  { type: "boolean", name: "True/False" },
];

const nums = [5, 10, 15, 20];

export default {
  name: "QuizApp",
  components: {
    LoadingApp,
    FooterNav,
  },
  data() {
    return {
      isStarted: false,
      categories: [],
      difficulties: difficulties,
      chosenCategory: null,
      chosenDifficulty: null,
      chosenType: null,
      types: types,
      nums: nums,
      chosenNum: 5,
      questions: [],
      chosenAnswers: [],
      currentQuestionIndex: 0,
      timer: 0,
      timerInterval: null,
      isSubmitted: false,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const url = "https://opentdb.com/api_category.php";

      axios
        .get(url)
        .then((response) => {
          if (!response.data.trivia_categories?.length) {
            return Promise.reject(response);
          }
          this.categories = response.data.trivia_categories.map((category) => ({
            ...category,
            name: category.name.replace(/\w+: /gi, ""),
          }));
        })
        .catch((error) => {
          console.error(error);
          alert(
            "Sorry, something went wrong trying to load the categories. Please try again."
          );
        });
    },
    setCategory(category) {
      this.chosenCategory = category;
    },
    setDifficulty(level) {
      this.chosenDifficulty = level;
    },
    setType(type) {
      this.chosenType = type;
    },
    setNum(num) {
      this.chosenNum = num;
    },
    stopTimer() {
      clearInterval(this.timerInterval);
    },
    startQuiz() {
      const url = this.generateUrl();
      this.isStarted = true;
      axios
        .get(url)
        .then((response) => {
          if (response.data.response_code !== 0) {
            return Promise.reject(response);
          }
          this.populateQuestions(response.data.results);
          this.setTimer();
        })
        .catch((error) => {
          console.error(error);
          alert(
            "Sorry, something went wrong trying to load the questions. Please try again."
          );
        });
    },
    generateUrl() {
      const difficultyParam = this.chosenDifficulty
        ? `&difficulty=${this.chosenDifficulty}`
        : "";
      const categoryParam = this.chosenCategory
        ? `&category=${this.chosenCategory}`
        : "";
      const typeParam = this.chosenType ? `&type=${this.chosenType}` : "";
      const numberParam = `&amount=${this.chosenNum}`;
      return `https://opentdb.com/api.php?${numberParam}${categoryParam}${difficultyParam}${typeParam}`;
    },
    populateQuestions(results) {
      this.questions = results.map((questionData) => {
        const answers = [
          ...questionData.incorrect_answers,
          questionData.correct_answer,
        ].map((answer) => ({
          text: answer,
          correct: answer === questionData.correct_answer,
        }));
        return {
          text: questionData.question,
          answers: this.shuffle(answers),
        };
      });
    },
    selectAnswer(index) {
      this.chosenAnswers[this.currentQuestionIndex] = index;
    },
    setTimer() {
      this.stopTimer();

      const numQuestions = this.chosenNum;
      let timePerQuestion;

      switch (this.chosenDifficulty) {
        case "easy":
          timePerQuestion = 5;
          break;
        case "medium":
          timePerQuestion = 8;
          break;
        case "hard":
          timePerQuestion = 12;
          break;
        default:
          timePerQuestion = 3;
      }

      this.timer = numQuestions * timePerQuestion;

      this.timerInterval = setInterval(() => {
        if (this.timer > 0) {
          this.timer--;
        } else {
          this.stopTimer();
        }
      }, 1000);
    },
    updateQuestionIndex(newIndex) {
      this.currentQuestionIndex = newIndex;
    },
    calcScore() {
      return this.questions.reduce((score, question, index) => {
        return (
          score + (question.answers[this.chosenAnswers[index]]?.correct ? 1 : 0)
        );
      }, 0);
    },
    completionMessage() {
      const score = this.calcScore();
      const total = this.questions.length;

      if (score === total) {
        return "Perfect score! Well done!";
      } else if (score / total > 0.7) {
        return "Great job! Keep it up!";
      } else {
        return "Nice try! Practice makes perfect!";
      }
    },
    reset() {
      this.isStarted = true;
      this.chosenAnswers = [];
      this.currentQuestionIndex = 0;
      this.setTimer();
      this.isSubmitted = false;
    },
    resetQuestions() {
      this.reset();
      this.startQuiz();
    },
    resetAll() {
      this.isStarted = false;
      this.questions = [];
      this.chosenAnswers = [];
      this.currentQuestionIndex = 0;
      this.timer = 0;
      this.stopTimer();
      this.isSubmitted = false;
    },
    shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    decodeHtml(html) {
      const text = document.createElement("textarea");
      text.innerHTML = html;
      return text.value;
    },
    answerCorrect(question, answerIndex) {
      return question.answers[answerIndex]?.correct || false;
    },
    submitQuiz() {
      this.isSubmitted = true;
      this.timer = 0;
    },
  },
};
</script>

<style lang="scss" scoped>
.quiz-setup {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  text-align: center;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  @media screen and (min-width: 768px) {
    padding: 3rem 1rem;
  }
  @media screen and (min-width: 1200px) {
    max-width: 70rem;
  }
  h1 {
    margin-top: 0;
  }

  h2 {
    font-size: 1.5rem;
    margin: 3rem auto 2rem;
    &::after {
      content: "";
      display: block;

      width: 100px;
      margin: 1rem auto 0;
    }
  }
}

.difficulty-options,
.categories-container,
.types-container {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
}

.difficulty-option,
.setup-option,
.type-option {
  flex: 1 1 auto;
  min-width: 120px;
  margin: 0.5rem;
  padding: 0.875rem;
  font-size: 1.125rem;
  color: #333;
  background-color: #f5f5f5;
  border: 2px solid #ddd;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  &:hover {
    color: #fff;
    background-color: $option-hover;
  }
  &.is-selected {
    background-color: $option-bg--active;
    color: #fff;
  }

  &:hover {
    color: #fff;
    background-color: #ef8156;
    border-color: #ef8156;
  }

  &.selected,
  &.is-selected {
    background-color: #ef8156;
    color: #fff;
    border-color: #ef8156;
  }
}

.button--start {
  margin-top: 3rem;
  color: #fff;
  background-color: #ef8156;
  border: 2px solid #ef8156;
  border-radius: 5px;
  padding: 0.875rem 1.5rem;
  text-transform: uppercase;
  font-weight: bold;
  transition: all 0.3s;

  &:hover {
    background-color: #e65c3c;
    border-color: #e65c3c;
  }
}

.restart-buttons {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 2rem;

  .button {
    margin: 1rem;
  }
}
.quiz {
  display: flex;
  flex-flow: row wrap;
  max-width: 40rem;
  width: 100%;
  & > * {
    flex: 1 1 100%;
  }
}
.answers-container {
  margin-top: 1rem;
  @media screen and (min-width: 768px) {
    margin-top: 3rem;
  }
}
.answer {
  margin-left: auto;
  margin-right: auto;
  font-size: 1.125rem;
  font-weight: 600;
  color: #313030;
  background-color: #fff;
  border-radius: 20px;
  &--option {
    max-width: 100%;
    padding: 1rem;
    cursor: pointer;
    transition: all 0.1s;
    &:hover {
      color: #fff;
      background-color: $option-hover;
    }
    &.is-selected {
      color: #fff;
      background-color: $option-bg--active;
    }
    & + * {
      margin-top: 0.75rem;
    }
  }
  &--correct,
  &--incorrect {
    margin-top: 0.75rem;
    margin-bottom: 0.25rem;
    padding: 1.25rem 1.5rem;
    color: #fff;
    @media screen and (min-width: 768px) {
      max-width: 50%;
    }
  }
  &--correct {
    background-color: #2da55d;
  }
  &--incorrect {
    background-color: #f8080c;
  }
}
.score {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 10rem;
  height: 10rem;
  margin: 0 auto;
  font-size: 3rem;
  font-weight: 600;
  border: 0.375rem solid;
  border-radius: 50%;
}
.completion-message {
  font-size: 2.5rem;
}
.quiz-answers {
  margin-top: 3rem;
  border-top: 2px solid;
}
.quiz-answer {
  margin: 4rem 2rem 6rem;
  .question {
    margin-bottom: 2.5rem;
    font-size: 1.5rem;
    &::after {
      content: "";
      display: block;
      border-bottom: 5px solid #fff;
      width: 100px;
      margin: 2rem auto 0;
    }
  }
}
.restart-buttons {
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  align-items: center;
  margin-top: 2rem;

  .button {
    margin-top: 3rem;
    color: #000000;
    background-color: #fff;
    border: 2px solid #fff;
    border-radius: 5px;
    padding: 0.875rem 1.5rem;
    text-transform: uppercase;
    font-weight: bold;
    transition: all 0.3s;

    &:hover {
      background-color: #e65c3c;
      border-color: #e65c3c;
    }
  }
  .timer {
    font-size: 1.25rem;
    font-weight: bold;
    margin: 1rem 0;
  }
}
</style>
