<template>
  <div>
    <div class="quiz-setup" v-if="!isStarted">
      <h1 >Start your Quiz</h1>
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

export default {
  name: 'QuizApp',
  data() {
    return {
      isStarted: false,
      categories: [],
      difficulties: difficulties,
      chosenCategory: null,
      chosenDifficulty: null,
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
.categories-container {
  display: flex;
  justify-content: center; 
  flex-wrap: wrap; /* Corrected from flex-flow: row wrap */
}

.difficulty-option,
.setup-option {
  flex: 1 1 auto;
  min-width: 120px; /* Increased min-width for better spacing */
  margin: 0.5rem;
  padding: 0.875rem;
  font-size: 1.125rem;
  color: #333; /* Changed to a darker color for better readability */
  background-color: #f5f5f5;
  border: 2px solid #ddd; /* Added border for clearer separation */
  border-radius: 8px; /* Rounded corners for a softer look */
  cursor: pointer;
  transition: all 0.3s; /* Smooth transition for hover effect */

  &:hover {
    color: #fff;
    background-color: #ef8156;
    border-color: #ef8156; /* Match border color with background on hover */
  }

  &.selected,
  &.is-selected {
    background-color: #ef8156;
    color: #fff;
    border-color: #ef8156; /* Ensure border color matches background when selected */
  }
}


.button--start {
  margin-top: 3rem;
  color: #fff;
  background-color: #ef8156;
  border: 2px solid #ef8156;  /* Add this line to create a border */
  border-radius: 5px; /* Optional: add border-radius for rounded corners */
  padding: 0.875rem 1.5rem; /* Optional: adjust padding for better appearance */
  text-transform: uppercase; /* Optional: make text uppercase */
  font-weight: bold; /* Optional: make text bold */
  transition: all 0.3s; /* Optional: smooth transition effect */
  
  &:hover {
    background-color: #e65c3c;
    border-color: #e65c3c; /* Ensure the border color changes on hover */
  }
}


</style>
