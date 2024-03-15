<template>
  <div id="app">
    <relation-graph></relation-graph>
  </div>
</template>

<script>
import RelationGraph from "@/components/relation-graph.vue";

export default {
  name: 'App',
  components: {
    RelationGraph,
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
  height: 100vh;
}
</style>
