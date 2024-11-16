<template>
  <div>Stopwatch</div>
  <div>{{ stopwatchDisplayedDuration }}</div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';

// Give the component a reactive version of Date.now() that we can update on
// an interval to make the template's stopwatch duration regularly rerender.
const now = ref(Date.now());
const updateNowValue = () => {
  now.value = Date.now();
}
let updateNowValueInterval: NodeJS.Timeout | null = null;
onMounted(() => {
  updateNowValueInterval = setInterval(updateNowValue, 1000);
});
onUnmounted(() => {
  clearInterval(updateNowValueInterval ?? undefined);
  updateNowValueInterval = null;
});

type Session = {
  startTime: number;
  stopTime: number | null;
}
const sessions = ref<Session[]>([]);
sessions.value.push({
  startTime: now.value,
  stopTime: null,
});

const stopwatchDisplayedDuration = computed(() => {
  const totalDuration = sessions.value.reduce<number>(
    (totalDuration, { startTime, stopTime }) => {
      const sessionDuration = (stopTime || now.value) - startTime;
      return totalDuration + sessionDuration;
    }, 0);

  const seconds = Math.floor((totalDuration / 1000) % 60).toString();
  const minutes = Math.floor((totalDuration / 1000 / 60) % 60).toString();
  const hours = Math.floor((totalDuration / 1000 / 60 / 60)).toString();

  const paddedSeconds = (seconds.length == 1 ? '0' : '') + seconds;
  const paddedMinutes = (minutes.length == 1 ? '0' : '') + minutes;
  const paddedHours = (hours.length == 1 ? '0' : '') + hours;

  return `${paddedHours}:${paddedMinutes}:${paddedSeconds}`;
});
</script>
