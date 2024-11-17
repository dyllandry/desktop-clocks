is<template>
  <div>Stopwatch</div>

  <div v-if="!isDurationEditorOpen">{{ formattedStopwatchDuration }}</div>

  <div v-if="isDurationEditorOpen">
    <input v-model="userInputEditedDuration" type="text" name="" id="">
  </div>

  <template v-if="!isDurationEditorOpen">
    <button v-if="!isRunning" @click="handleStartStopwatchPress">Start</button>
    <button v-if="isRunning" @click="stopStopwatch">Stop</button>
    <button @click="resetStopwatch">Reset</button>
    <button v-if="!isDurationEditorOpen" @click="openDurationEditor">Edit</button>
  </template>

  <template v-if="isDurationEditorOpen">
    <button @click="saveEditedDuration">Save</button>
    <button @click="closeDurationEditor">Cancel</button>
  </template>

  <div v-if="shownError" style="color: red;">{{shownError}}</div>

</template>

<script setup lang="ts">
import { computed, ref } from 'vue';

const shownError = ref<string | null>(null);

// By giving the component a reactive version of Date.now() and updating it
// regularly, we can make the shown stopwatch rerender as time passes.
const now = ref(Date.now());
const updateNowValue = () => {
  now.value = Date.now();
}
let updateNowValueInterval: NodeJS.Timeout | null = null;

type Session = {
  startTime: number;
  stopTime: number | null;
}
const sessions = ref<Session[]>([]);

const startStopwatch = (startTime?: number) => {
  sessions.value.push({
    startTime: startTime ?? Date.now(),
    stopTime: null,
  });
  updateNowValue();
  updateNowValueInterval = setInterval(updateNowValue, 1000);
};
const handleStartStopwatchPress = (_event: unknown) => startStopwatch();
const stopStopwatch = () => {
  const latestSession = sessions.value[sessions.value.length - 1];
  latestSession.stopTime = Date.now();
  clearInterval(updateNowValueInterval ?? undefined);
  updateNowValueInterval = null;
};
const resetStopwatch = () => {
  sessions.value = [];
};

const isDurationEditorOpen = ref(false);
const userInputEditedDuration = ref<null | string>(null);
const openDurationEditor = () => {
  if (isRunning.value) {
    stopStopwatch();
  }
  isDurationEditorOpen.value = true
  userInputEditedDuration.value = formattedStopwatchDuration.value;
}
const saveEditedDuration = () => {
  if (!userInputEditedDuration.value?.match(/\d\d:\d\d:\d\d/)) {
    shownError.value = `Invalid duration of ${userInputEditedDuration.value}. Please use the format 00:00:00.`;
    return;
  }
  const [hours, minutes, seconds] = userInputEditedDuration.value.split(":").map(s => parseInt(s));
  const hoursMs = hours * 60 * 60 * 1000;
  const minutesMs = minutes * 60 * 1000;
  const secondsMs = seconds * 1000;
  const totalMs = hoursMs + minutesMs + secondsMs;
  const neededStartingPointInPast = Date.now() - totalMs;
  closeDurationEditor();
  resetStopwatch();
  startStopwatch(neededStartingPointInPast);
  stopStopwatch();
};
const closeDurationEditor = () => {
  isDurationEditorOpen.value = false;
  if (shownError.value) {
    shownError.value = null;
  }
}

const isRunning = computed<boolean>(() => {
  const latestSession = sessions.value[sessions.value.length - 1];
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
