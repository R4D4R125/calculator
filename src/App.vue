<template>
  <div id="app">
   <app-calculator-insurance/>
  </div>
</template>

<script>
import AppCalculatorInsurance from './components/AppCalculatorInsurance.vue'

export default {
  name: 'App',
  components: {
    AppCalculatorInsurance,
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
}
</style>
