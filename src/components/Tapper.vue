<script setup lang="ts">
import { computed, ref, useTemplateRef } from 'vue';
import { filter, fromEvent, merge, scan, tap, timestamp } from 'rxjs';

const bpm = ref(0);
const hits = ref(0);
const resetProgressKey = ref(0);
const formattedBpm = computed(() => {
  const [whole, decimal] = bpm.value.toFixed(2).split('.');

  return { whole, decimal: `.${decimal}` };
});
const shouldPulsate = ref(false);
const showTapBorder = ref(false);
let tapBorderTimeout = 0;
let resetTimeoutId = 0;
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
const restartResetProgress = () => {
  resetProgressKey.value += 1;

  if (resetTimeoutId) {
    clearTimeout(resetTimeoutId);
  }

  resetTimeoutId = setTimeout(() => {
    if (hits.value === 1) {
      hits.value = 0;
    }
  }, resetTimeout);
};

// Keep keyboard and main-container taps in the same timing stream.
merge(
  fromEvent<KeyboardEvent>(document, 'keyup'),
  fromEvent<PointerEvent>(document, 'pointerup').pipe(
    // Document receives every pointer event, so only count events inside main.
    filter(
      (event) =>
        event.target instanceof Element && event.target.closest('main') !== null
    )
  )
)
  .pipe(
    tap(() => {
      showTapFeedback();
      restartResetProgress();
    }),
    // Add timestamps to each emission
    timestamp(),

    // Track the first tap as well as the intervals between later taps.
    scan(
      (acc, curr) => {
        if (
          acc.previousTimestamp === 0 ||
          curr.timestamp - acc.previousTimestamp >= resetTimeout
        ) {
          acc.time = 0;
          acc.hits = 1;
          acc.bpmValue = 0;
        } else {
          acc.time += curr.timestamp - acc.previousTimestamp;
          acc.hits += 1;
          acc.bpmValue = computeBPM(acc.time, acc.hits - 1);
        }

        acc.previousTimestamp = curr.timestamp;
        acc.hitsValue = acc.hits;

        return acc;
      },
      {
        time: 0,
        hits: 0,
        previousTimestamp: 0,
        bpmValue: 0,
        hitsValue: 0,
      }
    )
  )
  .subscribe(({ bpmValue, hitsValue }) => {
    bpm.value = bpmValue;
    hits.value = hitsValue;
    pulsate();
  });

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
    <div class="bpm-value">
      <span>{{ formattedBpm.whole }}</span
      ><span class="bpm-decimal">{{ formattedBpm.decimal }}</span>
    </div>
  </div>
  <div class="hits">
    <div>Hits: {{ hits }}</div>
    <div class="reset-progress" aria-hidden="true">
      <div
        :key="resetProgressKey"
        class="reset-progress-value"
        :class="{ active: resetProgressKey > 0 }"
      ></div>
    </div>
  </div>
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
  touch-action: none;
  -webkit-tap-highlight-color: transparent;
}

.bpm-label {
  font-family: 'Space Grotesk', 'Avenir Next', sans-serif;
  font-weight: 700;
}

.bpm-value {
  font-size: 2rem;
}

.bpm-decimal {
  color: #777;
  font-size: 0.6em;
}

.tap-border {
  border-color: #bb6666;
}

.hits {
  width: 200px;
  margin-top: 20px;
  font-family: 'Space Grotesk', 'Avenir Next', sans-serif;
  text-align: center;
  user-select: none;
}

.reset-progress {
  width: 100%;
  height: 4px;
  margin-top: 8px;
  overflow: hidden;
  border-radius: 4px;
}

.reset-progress-value {
  width: 100%;
  height: 100%;
  background-color: #efaaaa;
  transform-origin: left;
  transform: scaleX(0);
  border-radius: inherit;
}

.reset-progress-value.active {
  animation: reset-progress 2s linear forwards;
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

@keyframes reset-progress {
  from {
    transform: scaleX(1);
  }

  to {
    transform: scaleX(0);
  }
}
</style>
