<script setup>
import { computed, onBeforeUnmount, ref, watch } from "vue";

// --- CONFIGURACIÓN ---
const WORK_MINUTES = 25;
const WORK_SECONDS_DEFAULT = WORK_MINUTES * 60;

// --- ESTADO ---
const totalSeconds = ref(WORK_SECONDS_DEFAULT);
const isRunning = ref(false);
let timerId = null;

// --- Helpers ---
const setTitle = (t) => {
  if (typeof document !== "undefined") document.title = t;
};

// --- COMPUTED (UI) ---
const minutes = computed(() =>
  String(Math.floor(totalSeconds.value / 60)).padStart(2, "0")
);

const seconds = computed(() =>
  String(totalSeconds.value % 60).padStart(2, "0")
);

const statusText = computed(() => (isRunning.value ? "F O C U S" : "READY"));

// --- PROGRESS RING (SVG) ---
const RADIUS = 45;
const CIRCUMFERENCE = 2 * Math.PI * RADIUS;

const progressPercentage = computed(() => {
  return (totalSeconds.value / WORK_SECONDS_DEFAULT) * 100;
});

const circleDashoffset = computed(() => {
  return CIRCUMFERENCE - (progressPercentage.value / 100) * CIRCUMFERENCE;
});

// --- ACCIONES ---
function toggleTimer() {
  isRunning.value = !isRunning.value;
}

function reset() {
  isRunning.value = false;
  totalSeconds.value = WORK_SECONDS_DEFAULT;
  setTitle("Pomodoro");
}

// --- WATCH & LIFECYCLE ---
watch(isRunning, (running) => {
  // 🔒 seguridad: nunca dejes intervalos vivos
  if (timerId) clearInterval(timerId);
  timerId = null;

  if (running) {
    timerId = setInterval(() => {
      if (totalSeconds.value > 0) {
        totalSeconds.value -= 1;
        setTitle(`${minutes.value}:${seconds.value} - Pomodoro`);
      } else {
        isRunning.value = false; // disparará el watch y limpiará
        setTitle("Time's up!");
      }
    }, 1000);
  } else {
    setTitle("Pomodoro");
  }
});

onBeforeUnmount(() => {
  if (timerId) clearInterval(timerId);
});
</script>


<template>
  <main class="zen-container">

    <div class="glow-bg"></div>

    <section class="timer-wrapper">
      <h2 class="status-label">{{ statusText }}</h2>

      <div class="clock-display">
        <svg class="progress-ring" viewBox="0 0 100 100">
          <circle class="ring-circle-bg" cx="50" cy="50" r="45" />
          <circle class="ring-circle-fg" cx="50" cy="50" r="45"
            :style="{ strokeDashoffset: circleDashoffset, strokeDasharray: CIRCUMFERENCE }" />
        </svg>

        <div class="time-text">
          {{ minutes }}<span class="colon">:</span>{{ seconds }}
        </div>
      </div>

      <div class="controls">
        <button class="btn-main" :class="{ 'running': isRunning }" @click="toggleTimer">
          {{ isRunning ? 'Pausar' : 'Iniciar' }}
        </button>

        <button class="btn-reset" @click="reset" title="Reiniciar">
          <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
            stroke-linecap="round" stroke-linejoin="round">
            <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 12" />
          </svg>
        </button>
      </div>

      <p class="hint">MVP: Próximamente sonidos y tareas.</p>
    </section>
  </main>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Inter:wght@300;600&display=swap');

.zen-container {
  position: relative;
  min-height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #09090b;
  color: #ffffff;
  font-family: 'Inter', sans-serif;
  overflow: hidden;
}

/* --- CORRECCIÓN DEL EFECTO DE LUZ --- */
.glow-bg {
  position: absolute;
  width: 600px;
  height: 600px;
  /* El degradado ahora es más sutil */
  background: radial-gradient(circle, rgba(255, 107, 107, 0.15) 0%, rgba(9, 9, 11, 0) 60%);
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  pointer-events: none;
  z-index: 0;

  /* ESTAS DOS LÍNEAS ARREGLAN EL "CUADRADO" */
  border-radius: 50%;
  /* Vuelve el div redondo */
  filter: blur(80px);
  /* Difumina los bordes para que parezca gas/luz */
}

.timer-wrapper {
  position: relative;
  z-index: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
}

.status-label {
  font-size: 1rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.4);
  margin: 0;
  font-weight: 600;
}

.hint {
  margin-top: 1rem;
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.2);
}

.clock-display {
  position: relative;
  width: 300px;
  height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.time-text {
  font-family: 'JetBrains Mono', monospace;
  font-size: 5rem;
  font-weight: 700;
  font-variant-numeric: tabular-nums;
  z-index: 2;
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.1);
}

.colon {
  opacity: 0.5;
  margin: 0 5px;
  animation: blink 2s infinite;
}

@keyframes blink {

  0%,
  100% {
    opacity: 0.5;
  }

  50% {
    opacity: 0.2;
  }
}

.progress-ring {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  transform: rotate(-90deg);
  pointer-events: none;
}

.ring-circle-bg {
  fill: none;
  stroke: rgba(255, 255, 255, 0.05);
  stroke-width: 3;
}

.ring-circle-fg {
  fill: none;
  stroke: #ff6b6b;
  stroke-width: 3;
  stroke-linecap: round;
  stroke-dasharray: 283;
  stroke-dashoffset: 0;
  transition: stroke-dashoffset 1s linear;
  filter: drop-shadow(0 0 2px rgba(255, 107, 107, 0.5));
}

.controls {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-top: 10px;
}

.btn-main {
  background: #ffffff;
  color: #000;
  border: none;
  padding: 14px 40px;
  font-size: 1.1rem;
  font-weight: 600;
  border-radius: 50px;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 4px 15px rgba(255, 255, 255, 0.1);
}

.btn-main:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(255, 255, 255, 0.2);
}

.btn-main:active {
  transform: translateY(0);
}

.btn-main.running {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: none;
}

.btn-reset {
  background: transparent;
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.6);
  width: 50px;
  height: 50px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-reset:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

/* RESET GLOBAL */
:global(html),
:global(body) {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background-color: #09090b;
  overflow: hidden;
}

:global(#app) {
  width: 100vw;
  height: 100vh;
  max-width: 100% !important;
  margin: 0 !important;
  padding: 0 !important;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>