<script setup lang="ts">
import { ref, useTemplateRef } from 'vue';
import {
  filter,
  fromEvent,
  map,
  merge,
  pairwise,
  scan,
  tap,
  timestamp,
} from 'rxjs';

const bpm = ref(0);
const hits = ref(0);
const showTapBorder = ref(false);
let tapBorderTimeout: ReturnType<typeof setTimeout> | undefined;
const resetTimeout = 2000;
const computeBPM = (elapsed: number, hits: number) => {
  const seconds = elapsed / 1000;
  const avgTime = seconds / hits;

  return Math.round((60 / avgTime) * 100) / 100;
};
const showTapFeedback = () => {
  showTapBorder.value = true;

  if (tapBorderTimeout) {
    clearTimeout(tapBorderTimeout);
  }

  tapBorderTimeout = setTimeout(() => {
    showTapBorder.value = false;
  }, 75);
};

// Keep keyboard and main-container clicks in the same timing stream.
merge(
  fromEvent<KeyboardEvent>(document, 'keyup'),
  fromEvent<MouseEvent>(document, 'click').pipe(
    // Document receives every click, so only count clicks inside main.
    filter(
      (event) =>
        event.target instanceof Element && event.target.closest('main') !== null
    )
  )
)
  .pipe(
    tap(showTapFeedback),
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
            bpmValue: computeBPM(time, hits),
            // Adds the first hit swollen by the pairwise at beginning
            hitsValue: hits + 1,
          }
        : { bpmValue: 0, hitsValue: 0 };
    })
  )
  .subscribe(({ bpmValue, hitsValue }) => {
    bpm.value = bpmValue;
    hits.value = hitsValue;
    pulsate();
  });

const shouldPulsate = ref(false);
const tapperDiv = useTemplateRef<HTMLDivElement>('tapper');
const pulsate = () => {
  const MIN_COUNT_PULSE = 30;

  if (hits.value >= MIN_COUNT_PULSE) {
    // Sets the pulse animation to the same speed of the calculated BPM
    if (tapperDiv.value) {
      tapperDiv.value.style.animationDuration =
        (60 / bpm.value).toFixed(2) + 's';

      // Restart the one-shot pulse so it starts on the latest tap.
      if (shouldPulsate.value) {
        tapperDiv.value.classList.remove('pulse');
        // Force a layout recalculation before adding the class again.
        void tapperDiv.value.offsetWidth;
        tapperDiv.value.classList.add('pulse');
      }
    }

    // Enable animation after setting new duration
    shouldPulsate.value = true;
  } else {
    shouldPulsate.value = false;
  }
};
</script>

<template>
  <div
    ref="tapper"
    class="bpm-container"
    :class="{ pulse: shouldPulsate, 'tap-border': showTapBorder }"
  >
    <div class="bpm-label">BPM</div>
    <div class="bpm-value">{{ bpm.toFixed(2) }}</div>
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
  border: solid 4px transparent;
  transition: border-color 150ms ease;
}

.bpm-label {
  font-family: 'Space Grotesk', 'Avenir Next', sans-serif;
  font-weight: 700;
}

.bpm-value {
  font-size: 2rem;
}

.tap-border {
  border-color: #bb6666;
}

.pulse {
  animation: 0.25s pulse infinite linear;
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 15px rgba(248, 169, 120, 0.7);
  }

  100% {
    box-shadow: 0 0 0 0 rgba(248, 169, 120, 0);
  }
}
</style>
