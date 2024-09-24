<template>
  <Jeopardy_game/>
</template>

<script>
import Jeopardy_game from './components/jeopardy.vue'

export default {
  name: 'App',
  components: {
    Jeopardy_game
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
