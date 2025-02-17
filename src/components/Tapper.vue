<script setup lang="ts">
import { ref, useTemplateRef } from 'vue';
import { fromEvent, map, pairwise, scan, tap, timestamp } from 'rxjs';

const bpm = ref(0);
const hits = ref(0);
// Select the target element, e.g., the document
const target = document;
const resetTimeout = 2000;

// Create an Observable that emits events of a specific type, here 'keydown'
const keypress$ = fromEvent<KeyboardEvent>(target, 'keyup');
/* const tapperClick$ = fromEvent<MouseEvent>(
  document.querySelector('#tapper')!,
  'click'
);
 */
// Subscribe to the observable to handle key press events
keypress$
  .pipe(
    tap((event: Event) => pulseOnce(event)),

    // Step 1: Add timestamps to each emission
    timestamp(),

    // Step 2: Pair up consecutive emissions with their previous ones
    pairwise(),

    // Step 3: Compute the elapsed time between each pair of events
    map(([prev, curr]) => curr.timestamp - prev.timestamp),

    // Step 4: Calculate the average of all elapsed times
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
            bpmValue: Math.round(60e3 / (time / hits)),
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
const pulseOnce = (event: Event) => {
  event.stopPropagation();

  if (hits.value > 20) {
    shouldPulsate.value = true;

    // Sets the pulse animation to the same speet of the calculated BPM
    if (tapperDiv.value) {
      tapperDiv.value.style.animationDuration =
        (60 / bpm.value).toFixed(3) + 's';
    }
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
  background-color: #f8a978;
  color: #222;
  font-size: 1.5rem;
  border-radius: 50%;
  user-select: none;
  box-shadow: 0px 25px 50px -12px rgba(255, 255, 255, 0.25);
}

.pulse {
  animation: 0.25s pulse infinite;
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(255, 255, 255, 0.7);
  }

  70% {
    box-shadow: 0 0 0 15px rgba(255, 255, 255, 0);
  }

  100% {
    box-shadow: 0 0 0 0 rgba(255, 255, 255, 0);
  }
}
</style>
