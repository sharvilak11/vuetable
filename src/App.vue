<template>
  <div id="app">
    <NormalTable></NormalTable>
  </div>
</template>

<script>
import NormalTable from './examples/NormalTable';
export default {
  name: 'app',
  components: {
    NormalTable
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
