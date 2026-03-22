<template>
  <div class="app-container">
    <IntroPage v-if="currentView === 'intro'" @showRules="currentView = 'rules'" @playGame="currentView = 'game'" />
    <startPage v-else-if="currentView === 'rules'" @start="currentView = 'game'" />
    <Mastermind v-else-if="currentView === 'game'" />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import Mastermind from './components/Mastermind.vue'
import startPage from './components/startPage.vue'
import IntroPage from './components/IntroPage.vue'

export default defineComponent({
  name: 'App',
  components: {
    Mastermind,
    startPage,
    IntroPage,
  },
  setup() {
    const currentView = ref('intro')
    return { currentView }
  }
})
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;

  background-color: #82ddf0;
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  height: 100vh;
}
</style>
