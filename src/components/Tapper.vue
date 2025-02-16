<script setup lang="ts">
import { ref } from 'vue';
import { fromEvent, map, pairwise, scan, timestamp } from 'rxjs';

const bpm = ref('Tap');
// Select the target element, e.g., the document
const target = document;

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

    // Step 4: Calculate the average of all elapsed times once the sequence completes
    scan(
      (acc, timeDiff: number) => {
        acc.time += timeDiff;
        acc.hits++;

        return acc;
      },
      {
        time: 0,
        hits: 0,
      }
    ),

    map(({ time, hits }) => 60e3 / (time / hits))
  )
  .subscribe((time: number) => {
    // console.log(`BPM: ${time.toFixed(2)}`);
    bpm.value = time.toFixed(2);
  });
</script>

<template>
  <button>{{ bpm }}</button>
</template>

<style scoped></style>
