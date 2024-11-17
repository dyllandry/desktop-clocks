<template>
  <Stopwatch/>
</template>

<script setup lang="ts">
// I'm getting an error here: typescript: Cannot find module '/.Stopwatch.vue'
// or its corresponding type declarations. I'm not sure why. It doesn't happen
// with vscode. Doesn't seem to be messing with development yet.
// Github issue: https://github.com/vuejs/language-tools/issues/4883
import Stopwatch from './Stopwatch.vue';
</script>

<style>
body {
  background: black;
  color: white;
}
</style>
