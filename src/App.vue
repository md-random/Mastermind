<template>
  <Mastermind />
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import Mastermind from './components/Mastermind.vue'

export default defineComponent({
  name: 'App',
  components: {
    Mastermind,
  },
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background-color: rgb(88, 143, 143);
  height: 100vh;
}
</style>
