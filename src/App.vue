<template>

 
  <QuizApp></QuizApp>
</template>

<script>

import QuizApp from './components/QuizApp.vue';

export default {
  name: 'App',
  components: {
    
    QuizApp
  }
}
</script>
<style lang="scss">


#app {
  display: flex;
  align-items: center;
  justify-content: center; 
  min-height: 100vh; 
  padding: 2.5rem 0.625rem;
  text-align: center;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #fff;
  background-color: #1ca99f;
  letter-spacing: 0.5px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
