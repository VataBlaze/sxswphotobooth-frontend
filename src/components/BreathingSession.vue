<template>
  <div class="breathing-session absolute-full flex flex-center column">
    <!-- Phase indicator -->
    <div class="breathing-phase-label">
      {{ phaseLabel }}
    </div>

    <!-- Breathing circle -->
    <div class="breathing-circle-container">
      <div
        class="breathing-circle"
        :class="{ 'breathing-inhale': breathPhase === 'inhale', 'breathing-exhale': breathPhase === 'exhale', 'breathing-hold': breathPhase === 'hold' }"
      >
        <div class="breathing-circle-inner">
          <div class="breathing-instruction">
            {{ breathInstruction }}
          </div>
        </div>
      </div>
    </div>

    <!-- Timer -->
    <div class="breathing-timer">
      {{ formattedTimeRemaining }}
    </div>

    <!-- Progress bar -->
    <div class="breathing-progress-container">
      <div class="breathing-progress-bar" :style="{ width: progressPercent + '%' }"></div>
    </div>

    <!-- Skip button (small, subtle) -->
    <q-btn
      flat
      no-caps
      color="white"
      class="breathing-skip-btn"
      label="Skip →"
      @click="finish()"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

interface Props {
  durationSeconds?: number // total session duration
  inhaleSeconds?: number
  holdSeconds?: number
  exhaleSeconds?: number
}

const props = withDefaults(defineProps<Props>(), {
  durationSeconds: 180, // 3 minutes default
  inhaleSeconds: 4,
  holdSeconds: 4,
  exhaleSeconds: 6,
})

const emit = defineEmits<{
  sessionComplete: []
}>()

const timeRemaining = ref(props.durationSeconds)
const breathPhase = ref<'inhale' | 'hold' | 'exhale'>('inhale')
const breathInstruction = ref('Breathe in...')
const phaseLabel = ref('Close your eyes and relax')
let mainTimerId: number
let breathCycleTimerId: number

const cycleLength = computed(() => props.inhaleSeconds + props.holdSeconds + props.exhaleSeconds)

const formattedTimeRemaining = computed(() => {
  const mins = Math.floor(timeRemaining.value / 60)
  const secs = Math.floor(timeRemaining.value % 60)
  return `${mins}:${secs.toString().padStart(2, '0')}`
})

const progressPercent = computed(() => {
  return ((props.durationSeconds - timeRemaining.value) / props.durationSeconds) * 100
})

const runBreathCycle = () => {
  // Inhale
  breathPhase.value = 'inhale'
  breathInstruction.value = 'Breathe in...'

  breathCycleTimerId = window.setTimeout(() => {
    // Hold
    breathPhase.value = 'hold'
    breathInstruction.value = 'Hold...'

    breathCycleTimerId = window.setTimeout(() => {
      // Exhale
      breathPhase.value = 'exhale'
      breathInstruction.value = 'Breathe out...'

      breathCycleTimerId = window.setTimeout(() => {
        if (timeRemaining.value > 0) {
          runBreathCycle()
        }
      }, props.exhaleSeconds * 1000)
    }, props.holdSeconds * 1000)
  }, props.inhaleSeconds * 1000)
}

const finish = () => {
  cleanup()
  emit('sessionComplete')
}

const cleanup = () => {
  clearInterval(mainTimerId)
  clearTimeout(breathCycleTimerId)
}

onMounted(() => {
  // Start the main countdown
  mainTimerId = window.setInterval(() => {
    timeRemaining.value -= 1
    if (timeRemaining.value <= 0) {
      finish()
    }

    // Update phase label at intervals
    if (timeRemaining.value === props.durationSeconds - 5) {
      phaseLabel.value = 'Follow the rhythm'
    }
    if (timeRemaining.value <= 30) {
      phaseLabel.value = 'Almost done...'
    }
    if (timeRemaining.value <= 10) {
      phaseLabel.value = 'Get ready for your photo!'
    }
  }, 1000)

  // Start breath cycling
  runBreathCycle()
})

onBeforeUnmount(() => {
  cleanup()
})
</script>

<style lang="css">
.breathing-session {
  background: radial-gradient(ellipse at center, rgba(11, 5, 38, 0.92) 0%, rgba(5, 2, 18, 0.97) 100%);
  z-index: 10;
  user-select: none;
  pointer-events: auto;
  gap: 24px;
}

.breathing-phase-label {
  font-size: 1.4rem;
  color: rgba(255, 113, 206, 0.8);
  letter-spacing: 0.2em;
  text-transform: uppercase;
  text-align: center;
}

.breathing-circle-container {
  width: 45vmin;
  height: 45vmin;
  display: flex;
  align-items: center;
  justify-content: center;
}

.breathing-circle {
  width: 60%;
  height: 60%;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle, rgba(1, 205, 254, 0.3) 0%, rgba(185, 103, 255, 0.15) 60%, transparent 70%);
  box-shadow:
    0 0 60px rgba(1, 205, 254, 0.3),
    0 0 120px rgba(185, 103, 255, 0.15),
    inset 0 0 60px rgba(1, 205, 254, 0.1);
  transition: transform ease-in-out, box-shadow ease-in-out;
}

.breathing-circle.breathing-inhale {
  transform: scale(1.6);
  transition-duration: 4s;
  box-shadow:
    0 0 80px rgba(1, 205, 254, 0.5),
    0 0 160px rgba(185, 103, 255, 0.3),
    inset 0 0 80px rgba(1, 205, 254, 0.2);
}

.breathing-circle.breathing-hold {
  transform: scale(1.6);
  transition-duration: 0.3s;
  box-shadow:
    0 0 100px rgba(5, 255, 161, 0.4),
    0 0 180px rgba(1, 205, 254, 0.2),
    inset 0 0 60px rgba(5, 255, 161, 0.15);
}

.breathing-circle.breathing-exhale {
  transform: scale(1.0);
  transition-duration: 6s;
  box-shadow:
    0 0 40px rgba(185, 103, 255, 0.2),
    0 0 80px rgba(255, 113, 206, 0.1),
    inset 0 0 30px rgba(185, 103, 255, 0.05);
}

.breathing-circle-inner {
  text-align: center;
}

.breathing-instruction {
  font-size: 1.8rem;
  color: rgba(255, 251, 150, 0.9);
  text-shadow: 0 0 20px rgba(255, 251, 150, 0.5);
  letter-spacing: 0.05em;
}

.breathing-timer {
  font-size: 3rem;
  color: rgba(1, 205, 254, 0.7);
  font-variant-numeric: tabular-nums;
  text-shadow: 0 0 30px rgba(1, 205, 254, 0.3);
}

.breathing-progress-container {
  width: 60%;
  max-width: 400px;
  height: 3px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 2px;
  overflow: hidden;
}

.breathing-progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #FF71CE, #01CDFE, #B967FF);
  transition: width 1s linear;
  border-radius: 2px;
}

.breathing-skip-btn {
  position: absolute;
  bottom: 24px;
  right: 24px;
  opacity: 0.4;
  font-size: 0.9rem;
}
.breathing-skip-btn:hover {
  opacity: 0.8;
}
</style>
