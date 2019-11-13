<template>
  <div id="app">
    <Paper></Paper>
    <Slide></Slide>
  </div>
</template>

<script>
import Paper from '@/components/paper.vue'
import Slide from '@/components/slide.vue'

export default {
  name: 'app',
  components: {
    Paper,
    Slide
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
  width:300px;
  padding:10px;
  margin:0 auto;
  border:1px solid #ccc;
}
</style>
