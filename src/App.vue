<template>
  <div class="container">
    <Scene />
    <Panel />
  </div>
</template>

<script setup>
import Scene from "./components/Scene.vue";
import Panel from "./components/Panel.vue";
</script>

<style>
body {
  padding: 0;
  margin: 0;
}
.container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
}
</style>
