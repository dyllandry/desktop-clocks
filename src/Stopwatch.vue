<template>
  <div v-if="!isDurationEditorOpen" class="stopwatch-duration">{{ formattedStopwatchDuration }}</div>

  <input v-if="isDurationEditorOpen" v-model="userInputEditedDuration" type="text" ref="duration-editor" @keyup.enter="saveEditedDuration" class="stopwatch-duration" style="display: block;">

  <template v-if="!isDurationEditorOpen">
    <button v-if="!isRunning" @click="handleStartStopwatchPress">Start</button>
    <button v-if="isRunning" @click="stopStopwatch">Stop</button>
    <button v-if="!isDurationEditorOpen" @click="openDurationEditor" style="margin-left: 8px;">Edit</button>
  </template>

  <template v-if="isDurationEditorOpen">
    <button @click="saveEditedDuration" style="margin-right: 8px;">Save</button>
    <button @click="closeDurationEditor">Cancel</button>
  </template>

  <div v-if="shownError" style="color: red;">{{ shownError }}</div>

</template>

<script setup lang="ts">
import { computed, nextTick, ref, useTemplateRef } from 'vue';

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
const durationEditor = useTemplateRef<HTMLInputElement>('duration-editor');
const openDurationEditor = async () => {
  if (isRunning.value) {
    stopStopwatch();
  }
  isDurationEditorOpen.value = true
  userInputEditedDuration.value = formattedStopwatchDuration.value;

  // The nextTick() here is needed for focus to work. I'm not sure why.
  // This SO question has a comment that may help:
  // https://stackoverflow.com/q/56093602/7933478
  // The comment says calling ref.value.focus() directly manipulates the DOM,
  // and that the element may not be ready to receive focus immediately after
  // setting it to appear. When we mutate state in openDurationEditor() to show
  // the duration editor, we are already asking Vue to update the DOM. We should
  // wait for Vue to finish that first update before trying to focus the
  // element.
  await nextTick();
  durationEditor.value?.focus();
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
<style>
.stopwatch-duration {
  font-size: 20px;
}
</style>
