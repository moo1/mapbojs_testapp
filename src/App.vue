<template>
  <div id="app">
    <h3>Starbucks in Manhattan</h3>
    <Map1/>
  </div>
</template>

<script>
import Map1 from './components/Map1.vue'

export default {
  name: 'App',
  components: {
    Map1
  }
}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 20px;
}
</style>
