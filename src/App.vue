<template>
  <div class="draggable-top-bar"></div>
  <div style="margin: 0px 20px 20px 20px">
    <Stopwatch style="margin-bottom: 8px" />
    <button onclick="window.open()">New Stopwatch</button>
  </div>
</template>

<script setup lang="ts">
import Stopwatch from "./Stopwatch.vue";
</script>

<style>
body {
  background: black;
  color: white;
  margin: 0px;
  overflow: hidden;
}

.draggable-top-bar {
  height: 28px;
  width: 100%;
  app-region: drag;
}
</style>
