<script setup lang="ts">
import { ref } from 'vue';
import { fromEvent, map, pairwise, scan, timestamp } from 'rxjs';

const bpm = ref(0);
const hits = ref(0);
// Select the target element, e.g., the document
const target = document;
const resetTimeout = 2000;

// Create an Observable that emits events of a specific type, here 'keydown'
const keypress$ = fromEvent<KeyboardEvent>(target, 'keyup');

// Subscribe to the observable to handle key press events
keypress$
  .pipe(
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
</script>

<template>
  <div>BPM: {{ bpm }}</div>
  <div>Hits: {{ hits }}</div>
</template>

<style scoped></style>
