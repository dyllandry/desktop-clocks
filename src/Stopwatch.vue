<template>
  <div>Stopwatch</div>
  <div>{{ formattedStopwatchDuration }}</div>
  <button v-if="!isRunning" @click="startStopwatch">Start</button>
  <button v-if="isRunning" @click="stopStopwatch">Stop</button>
  <button @click="resetStopwatch">Reset</button>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue';

// By giving the component a reactive version of Date.now() and updating it
// regularly, we can make the shown stopwatch rerender as time passes.
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

const startStopwatch = () => {
  sessions.value.push({
    startTime: now.value,
    stopTime: null,
  });
};
const stopStopwatch = () => {
  const latestSession = sessions.value[sessions.value.length-1];
  latestSession.stopTime = Date.now();
};
const resetStopwatch = () => {
  sessions.value = [];
};

const isRunning = computed<boolean>(() => {
  const latestSession = sessions.value[sessions.value.length-1];
  return !!latestSession && latestSession.stopTime == null;
});

const formattedStopwatchDuration = computed(() => {
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
