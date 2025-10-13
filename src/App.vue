<script setup lang="ts">
import { ref, onMounted } from 'vue'
import SelectorButton from './components/SelectorButton.vue'
import StopButton from './components/StopButton.vue'
import SignalButton from './components/SignalButton.vue'

type Effect = {
  index: number
  id: number
}

const effectList = ref<Effect[]>([
  {
    index: 0,
    id: 0,
  },
  {
    index: 1,
    id: 1,
  },
  {
    index: 2,
    id: 2,
  },
  {
    index: 3,
    id: 3,
  },
  {
    index: 4,
    id: 4,
  },

  {
    index: 5,
    id: 5,
  },
  {
    index: 6,
    id: 6,
  },
  {
    index: 7,
    id: 7,
  },
  {
    index: 8,
    id: 8,
  },
])

const signalButtons = [
  { label: 'GREEN', id: 9, color: '#4CAF50' },
  { label: 'YELLOW', id: 10, color: '#FFEB3B' },
  { label: 'RED', id: 11, color: '#F44336' },
]

const selectedEffect = ref<null | Effect>(null)
const selectedSignalId = ref<number | null>(null)
const gain = ref<number>(0.5)
const signalDuration = ref<number>(0.5)

// 音声送信

const CONFIG = {
  BASE_FREQ: 18000, // 基本周波数
  FREQ_RANGE: 1000, // 周波数範囲
  NUM_CHANNELS: 16, // チャンネル数
  // SIGNAL_DURATION: 1.0,  // 信号長（秒）
  SAMPLE_RATE: 44100, // サンプリングレート
  FFT_SIZE: 8192, // FFT サイズ
  DETECTION_THRESHOLD: 0.3, // 検出閾値
}

// グローバル変数
let audioContext: AudioContext | null = null
let hostOscillator: OscillatorNode | null = null
let hostGain: GainNode | null = null
let isHostActive = false
let interval: number | null = null

function getFrequencyForChannel(channel: number) {
  const step = CONFIG.FREQ_RANGE / CONFIG.NUM_CHANNELS
  return CONFIG.BASE_FREQ + (CONFIG.NUM_CHANNELS - channel) * step
}

async function initializeHost() {
  try {
    audioContext = new window.AudioContext()

    // ゲインノードを作成
    hostGain = audioContext.createGain()
    hostGain.connect(audioContext.destination)
    hostGain.gain.setValueAtTime(0, audioContext.currentTime)

    isHostActive = true
  } catch (error) {
    console.error('ホスト初期化エラー:', error)
  }
}

function delay(ms: number): Promise<boolean> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(true)
    }, ms)
  })
}

// エフェクト送信
async function sendEffect(effectId: number, duration: number) {
  if (!isHostActive || !audioContext) return

  const frequency = getFrequencyForChannel(effectId)

  let now = audioContext.currentTime

  if (hostOscillator) {
    hostGain!.gain.cancelScheduledValues(now)
    hostGain!.gain.setValueAtTime(hostGain!.gain.value, now)
    hostGain!.gain.linearRampToValueAtTime(0, now + 0.01)
    hostOscillator.stop(now + 0.01)
  } else {
    hostGain!.gain.cancelScheduledValues(now)
    hostGain!.gain.setValueAtTime(0, now)
  }

  // 既存のオシレーターを停止

  const wait = await delay(10)

  if (wait) {
    if (interval) clearInterval(interval)
    const retfn = () => {
      hostOscillator = audioContext!.createOscillator()
      hostOscillator.type = 'sine'
      hostOscillator.frequency.setValueAtTime(frequency, audioContext!.currentTime)
      hostOscillator.connect(hostGain!)
      // エンベロープを設定（フェードイン・フェードアウト）
      now = audioContext!.currentTime
      hostGain!.gain.setValueAtTime(0, now)
      hostGain!.gain.linearRampToValueAtTime(gain.value, now + 0.01)
      hostGain!.gain.setValueAtTime(gain.value, now + duration - 0.01)
      hostGain!.gain.linearRampToValueAtTime(0, now + duration)

      // 信号を送信
      hostOscillator.start(now)
      hostOscillator.stop(now + duration)

      // UI更新

      console.log(`エフェクト送信: ID=${effectId}, 周波数=${frequency}Hz`)
    }

    interval = setInterval(
      () => {
        if (typeof retfn === 'function') retfn()
      },
      duration * 1000 * 1.5,
    )
  }
  // 新しいオシレーターを作成
}

function stopEffect() {
  if (!isHostActive || !audioContext) return

  selectedEffect.value = null
  selectedSignalId.value = null

  const now = audioContext.currentTime

  if (hostOscillator) {
    hostGain!.gain.cancelScheduledValues(now)
    hostGain!.gain.setValueAtTime(hostGain!.gain.value, now)
    hostGain!.gain.linearRampToValueAtTime(0, now + 0.01)
    hostOscillator.stop(now + 0.01)
  } else {
    hostGain!.gain.cancelScheduledValues(now)
    hostGain!.gain.setValueAtTime(0, now)
  }

  clearInterval(interval!)
}

onMounted(() => {
  initializeHost()
})

function onSelectorButtonClick(effect: Effect) {
  selectedEffect.value = effect
  selectedSignalId.value = null
  if (audioContext) {
    sendEffect(effect.id, signalDuration.value)
  }
}

function onSignalButtonClick(id: number) {
  selectedSignalId.value = id
  selectedEffect.value = null
  if (audioContext) {
    sendEffect(id, signalDuration.value)
  }
}

function resetGain() {
  gain.value = 0.5
}

function resetDuration() {
  signalDuration.value = 0.5
}
</script>

<template>
  <div class="app-container">
    <div class="main-layout">
      <!-- Left Panel -->
      <div class="left-panel">
        <!-- Color Selector -->
        <div class="color-selector">
          <SignalButton
            v-for="signal in signalButtons"
            :key="signal.id"
            :label="signal.label"
            :id="signal.id"
            :color="signal.color"
            :isSelected="selectedSignalId === signal.id"
            @select="onSignalButtonClick"
          />
        </div>

        <!-- Sliders -->
        <div class="sliders-container">
          <div class="slider-group">
            <div class="slider-header">
              <span class="slider-label">GAIN {{ gain }}</span>
              <button class="reset-button" @click="resetGain">↻</button>
            </div>
            <input
              id="gain"
              type="range"
              step="0.1"
              min="0"
              max="1"
              v-model.number="gain"
              class="slider"
            />
          </div>

          <div class="slider-group">
            <div class="slider-header">
              <span class="slider-label">DURATION {{ signalDuration }}</span>
              <button class="reset-button" @click="resetDuration">↻</button>
            </div>
            <input
              id="duration"
              type="range"
              step="0.1"
              min="0.2"
              max="1"
              v-model.number="signalDuration"
              class="slider"
            />
          </div>
        </div>

        <!-- Stop Button -->
        <div class="stop-button-container">
          <StopButton
            :isSelected="!selectedEffect && !selectedSignalId"
            @stop="stopEffect()"
          ></StopButton>
        </div>
      </div>

      <!-- Right Panel - Effects Grid -->
      <div class="right-panel">
        <div class="effects-grid">
          <SelectorButton
            v-for="effect in effectList"
            :key="effect.index"
            :id="effect.id"
            :index="effect.index"
            :isSelected="selectedEffect?.index === effect.index || selectedSignalId === effect.id"
            @select="onSelectorButtonClick(effect)"
          >
          </SelectorButton>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
/* Global reset */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
</style>

<style scoped>
/* Dark theme base */
.app-container {
  background-color: #000;
  color: #fff;
  width: 100vw;
  height: 100vh;
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.main-layout {
  display: flex;
  height: 100vh;
}

/* Left Panel */
.left-panel {
  background-color: #111;
  width: 300px;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  border-right: 1px solid #333;
  height: 100vh;
  box-sizing: border-box;
}

/* Color Selector */
.color-selector {
  display: flex;
  flex-direction: row;
  gap: 10px;
  background-color: #222;
  padding: 15px;
  border-radius: 10px;
  border: 1px solid #333;
}

/* Sliders */
.sliders-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.slider-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.slider-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.slider-label {
  color: #fff;
  font-weight: bold;
  font-size: 14px;
}

.reset-button {
  background-color: transparent;
  border: 1px solid #fff;
  color: #fff;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
}

.reset-button:hover {
  background-color: #333;
}

.slider {
  -webkit-appearance: none;
  appearance: none;
  height: 6px;
  background: #333;
  outline: none;
  border-radius: 3px;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  background: #fff;
  border-radius: 50%;
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  background: #fff;
  border-radius: 50%;
  cursor: pointer;
  border: none;
}

/* Stop Button */
.stop-button-container {
  margin-top: auto;
  display: flex;
  justify-content: center;
}

/* Right Panel */
.right-panel {
  flex: 1;
  padding: 20px;
  background-color: #000;
  height: 100vh;
  box-sizing: border-box;
}

.effects-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 10px;
  height: 100%;
}

/* Legacy styles for compatibility */
.flex-row {
  display: flex;
  flex-direction: row;
  gap: 10px;
  align-items: center;
}

.flex-column {
  display: flex;
  flex-direction: column;
  gap: 5px;
}
</style>
