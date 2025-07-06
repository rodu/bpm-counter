<script setup lang="ts">
import { ref, useTemplateRef } from 'vue';
import { fromEvent, map, pairwise, scan, tap, timestamp } from 'rxjs';

const bpm = ref(0);
const hits = ref(0);
const resetTimeout = 2000;

// Create an Observable that emits events of a type 'keyup'
fromEvent<KeyboardEvent>(document, 'keyup')
  .pipe(
    tap((event: Event) => pulsate(event)),

    // Add timestamps to each emission
    timestamp(),

    // Pair up consecutive emissions with their previous ones
    pairwise(),

    // Compute the elapsed time between each pair of events
    map(([prev, curr]) => curr.timestamp - prev.timestamp),

    // Calculate the average of all elapsed times
    scan(
      (acc, elapsed: number) => {
        if (elapsed >= resetTimeout) {
          // Resets counters
          return { time: 0, hits: 0 };
        }

        acc.time += elapsed;
        acc.hits += 1;

        return acc;
      },
      {
        time: 0,
        hits: 0,
      }
    ),

    map(({ time, hits }) => {
      return hits > 0
        ? {
            bpmValue: Number.parseFloat((60e3 / (time / hits)).toFixed(2)),
            hitsValue: hits,
          }
        : { bpmValue: 0, hitsValue: 0 };
    })
  )
  .subscribe(({ bpmValue, hitsValue }) => {
    bpm.value = bpmValue;
    hits.value = hitsValue;
  });

const shouldPulsate = ref(false);
const tapperDiv = useTemplateRef('tapper');
const pulsate = (event: Event) => {
  event.stopPropagation();
  const MIN_COUNT_PULSE = 40;

  if (hits.value >= MIN_COUNT_PULSE) {
    // Sets the pulse animation to the same speet of the calculated BPM
    if (tapperDiv.value) {
      tapperDiv.value.style.animationDuration =
        (60 / bpm.value).toFixed(3) + 's';
    }

    // Enable animation after setting new duration
    shouldPulsate.value = true;
  } else {
    shouldPulsate.value = false;
  }
};
</script>

<template>
  <div ref="tapper" class="bpm-container" :class="{ pulse: shouldPulsate }">
    <div>BPM</div>
    <div>{{ bpm }}</div>
  </div>
  <div>Hits: {{ hits }}</div>
</template>

<style scoped>
.bpm-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 200px;
  height: 200px;
  background-color: rgb(248, 169, 120);
  color: #222;
  font-size: 1.5rem;
  border-radius: 50%;
  user-select: none;
  box-shadow: 0px 25px 50px -12px rgba(248, 169, 120, 0.25);
}

.pulse {
  animation: 0.25s pulse infinite;
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 10px rgba(248, 169, 120, 0.7);
  }

  50% {
    box-shadow: 0 0 0 15px rgba(248, 169, 120, 0);
  }

  100% {
    box-shadow: 0 0 0 0 rgba(248, 169, 120, 0);
  }
}
</style>
