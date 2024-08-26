<template>
  <div>
    <div class="quiz-setup" v-if="!isStarted">
      <h1>Start your Quiz</h1>
      <h2>Difficulty</h2>
      <div class="difficulty-options">
        <div
          class="difficulty-option"
          :class="{ 'selected': chosenDifficulty == difficulty.level }"
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
          :class="{ 'is-selected': chosenCategory == null }"
          @click="setCategory(null)"
        >
          Any Category
        </div>
        <div
          class="setup-option"
          :class="{ 'is-selected': chosenCategory == category.id }"
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
          :class="{ 'is-selected': chosenType == type.type }"
          v-for="(type, index) in types"
          :key="index"
          @click="setType(type.type)"
        >
          {{ type.name }}
        </div>
      </div>

      <button class="button button--start" type="button" @click="startQuiz()">Start</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const difficulties = [
  { level: null, name: 'Any Difficulty' },
  { level: 'easy', name: 'Easy' },
  { level: 'medium', name: 'Medium' },
  { level: 'hard', name: 'Hard' },
];

const types = [
  { type: null, name: 'Any Type' },
  { type: 'multiple', name: 'Multiple Choice' },
  { type: 'boolean', name: 'True/False' },
];

export default {
  name: 'QuizApp',
  data() {
    return {
      isStarted: false,
      categories: [],
      difficulties: difficulties,
      chosenCategory: null,
      chosenDifficulty: null,
      chosenType: null,
      types: types,
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const url = 'https://opentdb.com/api_category.php';

      axios
        .get(url)
        .then((response) => {
          if (!response.data.trivia_categories?.length) {
            return Promise.reject(response);
          }
          this.categories = response.data.trivia_categories.map((category) => ({
            ...category,
            name: category.name.replace(/\w+: /gi, ''),
          }));
        })
        .catch((error) => {
          console.error(error);
          alert('Sorry, something went wrong trying to load the categories. Please try again.');
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
    startQuiz() {
      this.isStarted = true;
      // Logic to start the quiz can go here
    },
    generateUrl() {
      const difficultyParam = this.chosenDifficulty ? `&difficulty=${this.chosenDifficulty}` : '';
      const categoryParam = this.chosenCategory ? `&category=${this.chosenCategory}` : '';
      const typeParam = this.chosenType ? `&type=${this.chosenType}` : '';

      return `https://opentdb.com/api.php?${categoryParam}${difficultyParam}${typeParam}`;
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

  h1 {
    margin-top: 0;
  }

  h2 {
    font-size: 1.5rem;
    margin: 3rem auto 2rem;

    &::after {
      content: '';
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
</style>
