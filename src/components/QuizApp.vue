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
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: "QuizApp",
    data() {
      return {
        isStarted: false,
        chosenDifficulty: null,
        difficulties: [
          { level: null, name: "Any Difficulty" },
          { level: "easy", name: "Easy" },
          { level: "medium", name: "Medium" },
          { level: "hard", name: "Hard" },
        ],
      };
    },
    methods: {
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
        content: "";
        display: block;
        width: 100px;
        margin: 1rem auto 0;
      }
    }
  }
  .difficulty-options {
    display: flex;
    justify-content: center; 
    flex-flow: row wrap;
  }
  .difficulty-option {
    flex: 1 1 auto;
    min-width: 100px;
    margin: 0.5rem;
    padding: 0.875rem;
    font-size: 1.125rem;
    color: #313030;
    background-color: #f5f5f5;
    border-radius: 1px;
    cursor: pointer;
    transition: all 0.1s;
    &:hover {
      color: #fff;
      background-color: #ef8156;
    }
    &.selected {
      background-color: #ef8156;
      color: #fff;
    }
  }
  </style>
  
  