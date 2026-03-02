<script setup>
import { ref, computed } from 'vue'

const segments = [
  { label: '10 ball', color: '#e74c3c' },
  { label: '4 ball', color: '#3498db' },
  { label: '3 ball', color: '#27ae60' },
  { label: "supper o'yin", color: '#9b59b6' },
  { label: '2 ball', color: '#e67e22' },
  { label: '3 ball', color: '#1abc9c' },
  { label: '4 ball', color: '#c0392b' },
  { label: '3 ball', color: '#2980b9' },
  { label: '1 ball', color: '#16a085' },
  { label: '2 ball', color: '#f39c12' },
  { label: "Bo'sh yo'lak", color: '#8e44ad' },
  { label: "Bo'sh yo'lak", color: '#d35400' },
  { label: 'Respublikalar soni', color: '#e74c3c' },
  { label: 'monarxiya davlatlari', color: '#34495e' },
  { label: 'afrikada nechta davlat bor', color: '#3498db' },
  { label: 'yevropa nechta davlat bor', color: '#9b59b6' },
  { label: 'jahonda nechta davlat bor', color: '#e67e22' },
  { label: 'nechta materik mavjud', color: '#1abc9c' },
  { label: 'Avstriya poytaxti', color: '#c0392b' },
  { label: 'fransiya poytaxti qayer', color: '#2980b9' },
  { label: 'Daniya poytaxti', color: '#27ae60' },
  { label: 'Albaniya poytaxti', color: '#8e44ad' },
  { label: '5 ball', color: '#e74c3c' },
  { label: 'belgiya poytaxti', color: '#16a085' },
]

const N = segments.length
const SEG_ANGLE = 360 / N
const CX = 250
const CY = 250
const R = 235
const IR = 45
const TEXT_R = IR + (R - IR) * 0.55

const spinning = ref(false)
const rotation = ref(0)
const spinsLeft = ref(3)
const results = ref([])
const used = ref(new Set())
const lastResult = ref(null)
const showResult = ref(false)

function polar(r, angleDeg) {
  const rad = (angleDeg - 90) * Math.PI / 180
  return { x: CX + r * Math.cos(rad), y: CY + r * Math.sin(rad) }
}

function segPath(i) {
  const a1 = i * SEG_ANGLE
  const a2 = (i + 1) * SEG_ANGLE
  const o1 = polar(R, a1), o2 = polar(R, a2)
  const i1 = polar(IR, a1), i2 = polar(IR, a2)
  return `M${i1.x},${i1.y} L${o1.x},${o1.y} A${R},${R} 0 0 1 ${o2.x},${o2.y} L${i2.x},${i2.y} A${IR},${IR} 0 0 0 ${i1.x},${i1.y}Z`
}

function textAngle(i) {
  return (i + 0.5) * SEG_ANGLE - 90
}

function textSize(label) {
  if (label.length > 22) return 7
  if (label.length > 16) return 8
  return 9.5
}

function segColor(i) {
  return used.value.has(i) ? '#444' : segments[i].color
}

const wheelStyle = computed(() => ({
  transform: `rotate(${rotation.value}deg)`,
}))

function spin() {
  if (spinning.value || spinsLeft.value <= 0) return
  showResult.value = false

  const available = []
  for (let i = 0; i < N; i++) {
    if (!used.value.has(i)) available.push(i)
  }
  if (!available.length) return

  const target = available[Math.floor(Math.random() * available.length)]
  const offset = (Math.random() - 0.5) * SEG_ANGLE * 0.6
  const targetAngle = (target + 0.5) * SEG_ANGLE + offset
  const land = ((360 - targetAngle) % 360 + 360) % 360

  const minRot = rotation.value + 360 * 5
  const k = Math.ceil((minRot - land) / 360)
  const newRot = k * 360 + land

  spinning.value = true

  requestAnimationFrame(() => {
    requestAnimationFrame(() => {
      rotation.value = newRot

      setTimeout(() => {
        spinning.value = false
        spinsLeft.value--
        used.value = new Set([...used.value, target])
        lastResult.value = segments[target]
        results.value.push({ ...segments[target], index: target })
        showResult.value = true
      }, 5200)
    })
  })
}

function reset() {
  spinning.value = false
  showResult.value = false
  lastResult.value = null
  results.value = []
  used.value = new Set()
  spinsLeft.value = 3
  rotation.value = 0
}
</script>

<template>
  <div class="game">
    <h1 class="title">Baraban</h1>

    <div class="spins-info">
      <span
        v-for="i in 3"
        :key="i"
        class="dot"
        :class="{ active: i <= spinsLeft }"
      ></span>
      <span class="spins-text">{{ spinsLeft }} / 3 aylantirish</span>
    </div>

    <div class="wheel-area">
      <!-- Pointer -->
      <div class="pointer-wrap">
        <div class="pointer"></div>
      </div>

      <div class="wheel" :class="{ spinning: spinning }" :style="wheelStyle">
        <svg viewBox="0 0 500 500">
          <circle
            :cx="CX" :cy="CY" :r="R + 3"
            fill="none"
            stroke="rgba(255,255,255,0.15)"
            stroke-width="5"
          />

          <g v-for="(seg, i) in segments" :key="i">
            <path
              :d="segPath(i)"
              :fill="segColor(i)"
              stroke="rgba(255,255,255,0.2)"
              stroke-width="1"
            />
            <g :transform="`rotate(${textAngle(i)}, ${CX}, ${CY})`">
              <text
                :x="CX + TEXT_R"
                :y="CY"
                text-anchor="middle"
                dominant-baseline="central"
                fill="white"
                :font-size="textSize(seg.label)"
                font-weight="700"
                font-family="sans-serif"
                :opacity="used.has(i) ? 0.25 : 1"
              >{{ seg.label }}</text>
            </g>
          </g>

          <circle
            :cx="CX" :cy="CY" :r="IR - 3"
            fill="#0f0f1a"
            stroke="rgba(255,255,255,0.12)"
            stroke-width="3"
          />
          <circle :cx="CX" :cy="CY" r="18" fill="#181830" />
        </svg>
      </div>
    </div>

    <button
      class="spin-btn"
      @click="spin"
      :disabled="spinning || spinsLeft <= 0"
    >
      {{ spinning ? 'Aylanmoqda...' : spinsLeft > 0 ? 'Aylantirish' : 'Tugadi!' }}
    </button>

    <Transition name="pop">
      <div v-if="showResult && lastResult" class="result-popup">
        <div class="result-badge" :style="{ borderColor: lastResult.color, color: lastResult.color }">
          {{ lastResult.label }}
        </div>
      </div>
    </Transition>

    <div v-if="results.length" class="history">
      <h3 class="history-title">Natijalar</h3>
      <div v-for="(r, i) in results" :key="i" class="history-item">
        <span class="history-num">{{ i + 1 }}</span>
        <span class="history-label" :style="{ background: r.color }">{{ r.label }}</span>
      </div>
    </div>

    <button
      v-if="spinsLeft <= 0 && !spinning"
      class="reset-btn"
      @click="reset"
    >
      Qaytadan boshlash
    </button>
  </div>
</template>

<style scoped>
.game {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 20px 16px 40px;
  max-width: 620px;
  margin: 0 auto;
}

.title {
  font-size: 2.4rem;
  font-weight: 800;
  margin: 0;
  background: linear-gradient(135deg, #e74c3c, #f39c12, #2ecc71, #3498db, #9b59b6);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: 3px;
  text-transform: uppercase;
}

.spins-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.dot {
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #333;
  border: 2px solid #555;
  transition: all 0.4s;
}

.dot.active {
  background: #2ecc71;
  border-color: #27ae60;
  box-shadow: 0 0 10px rgba(46, 204, 113, 0.5);
}

.spins-text {
  font-size: 1.05rem;
  color: #888;
  margin-left: 6px;
  font-weight: 500;
}

.wheel-area {
  position: relative;
  width: min(480px, 88vw);
  height: min(480px, 88vw);
}

.pointer-wrap {
  position: absolute;
  top: -8px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
}

.pointer {
  width: 0;
  height: 0;
  border-left: 16px solid transparent;
  border-right: 16px solid transparent;
  border-top: 32px solid #ffd700;
  filter: drop-shadow(0 3px 6px rgba(0, 0, 0, 0.6));
}

.wheel {
  width: 100%;
  height: 100%;
  will-change: transform;
}

.wheel.spinning {
  transition: transform 5s cubic-bezier(0.15, 0.85, 0.05, 1);
}

.wheel svg {
  width: 100%;
  height: 100%;
  filter: drop-shadow(0 6px 30px rgba(0, 0, 0, 0.5));
}

.spin-btn {
  padding: 16px 52px;
  font-size: 1.2rem;
  font-weight: 700;
  border: none;
  border-radius: 50px;
  background: linear-gradient(135deg, #e74c3c, #c0392b);
  color: white;
  cursor: pointer;
  transition: all 0.3s;
  text-transform: uppercase;
  letter-spacing: 2px;
  box-shadow: 0 4px 15px rgba(231, 76, 60, 0.3);
}

.spin-btn:hover:not(:disabled) {
  transform: translateY(-2px) scale(1.03);
  box-shadow: 0 6px 25px rgba(231, 76, 60, 0.5);
}

.spin-btn:active:not(:disabled) {
  transform: translateY(0) scale(0.98);
}

.spin-btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
  box-shadow: none;
}

.result-popup {
  text-align: center;
}

.result-badge {
  display: inline-block;
  padding: 14px 36px;
  border-radius: 14px;
  border: 3px solid;
  background: rgba(255, 255, 255, 0.04);
  font-size: 1.5rem;
  font-weight: 800;
  letter-spacing: 1px;
}

.pop-enter-active {
  animation: popIn 0.45s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.pop-leave-active {
  animation: popIn 0.25s ease reverse;
}

@keyframes popIn {
  0% {
    transform: scale(0.4);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.history {
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
  max-width: 400px;
}

.history-title {
  font-size: 1rem;
  color: #888;
  font-weight: 600;
  margin: 0;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.history-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.history-num {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.08);
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.85rem;
  flex-shrink: 0;
  color: #ccc;
}

.history-label {
  padding: 7px 18px;
  border-radius: 22px;
  font-weight: 600;
  font-size: 0.95rem;
  color: white;
}

.reset-btn {
  padding: 12px 36px;
  font-size: 1rem;
  font-weight: 600;
  border: 2px solid #3498db;
  border-radius: 50px;
  background: transparent;
  color: #3498db;
  cursor: pointer;
  transition: all 0.3s;
  letter-spacing: 1px;
}

.reset-btn:hover {
  background: #3498db;
  color: white;
  box-shadow: 0 4px 15px rgba(52, 152, 219, 0.3);
}
</style>
