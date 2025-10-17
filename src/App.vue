<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import SelectorButton from './components/SelectorButton.vue'
import StopButton from './components/StopButton.vue'
import SignalButton from './components/SignalButton.vue'
import BackgroundCanvas from './components/BackgroundCanvas.vue'
import IconRefresh from '~icons/mdi/refresh'

type Effect = {
  index: number
  id: number
  name: string
  imageUrl?: string
}

// BLUE用エフェクトリスト (ID: 0-8)
const blueEffectList = ref<Effect[]>([
  {
    index: 0,
    id: 0,
    name: 'Arrow',
    imageUrl: 'https://via.placeholder.com/200x150/ff0000/ffffff?text=Red+Blob',
  },
  {
    index: 1,
    id: 1,
    name: 'Effect name',
    imageUrl: 'https://via.placeholder.com/200x150/ffffff/000000?text=A+Pattern',
  },
  {
    index: 2,
    id: 2,
    name: 'Effect name',
    imageUrl: 'https://via.placeholder.com/200x150/000000/ffffff?text=Effect+3',
  },
  {
    index: 3,
    id: 3,
    name: 'Effect name',
    imageUrl: 'https://via.placeholder.com/200x150/000000/ffffff?text=Effect+4',
  },
  {
    index: 4,
    id: 4,
    name: 'Effect name',
    imageUrl: 'https://via.placeholder.com/200x150/000000/ffffff?text=Effect+5',
  },
  {
    index: 5,
    id: 5,
    name: 'Effect name',
    imageUrl: 'https://via.placeholder.com/200x150/000000/ffffff?text=Effect+6',
  },
  {
    index: 6,
    id: 6,
    name: 'Star Filter',
    imageUrl: 'public/star_filter.png',
  },
  {
    index: 7,
    id: 7,
    name: 'No effect',
    imageUrl: 'https://via.placeholder.com/200x150/000000/ffffff?text=No+Effect',
  },
  {
    index: 8,
    id: 8,
    name: 'No effect',
    imageUrl: 'https://via.placeholder.com/200x150/000000/ffffff?text=No+Effect',
  },
])

// YELLOW用エフェクトリスト (ID: 12-20)
const yellowEffectList = ref<Effect[]>([
  {
    index: 0,
    id: 12,
    name: 'Yellow Effect 1',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+1',
  },
  {
    index: 1,
    id: 13,
    name: 'Yellow Effect 2',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+2',
  },
  {
    index: 2,
    id: 14,
    name: 'Yellow Effect 3',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+3',
  },
  {
    index: 3,
    id: 15,
    name: 'Yellow Effect 4',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+4',
  },
  {
    index: 4,
    id: 16,
    name: 'Yellow Effect 5',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+5',
  },
  {
    index: 5,
    id: 17,
    name: 'Yellow Effect 6',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+6',
  },
  {
    index: 6,
    id: 18,
    name: 'Yellow Effect 7',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+7',
  },
  {
    index: 7,
    id: 19,
    name: 'Yellow Effect 8',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+8',
  },
  {
    index: 8,
    id: 20,
    name: 'Yellow Effect 9',
    imageUrl: 'https://via.placeholder.com/200x150/ffeb3b/000000?text=Yellow+9',
  },
])

// RED用エフェクトリスト (ID: 21-29)
const redEffectList = ref<Effect[]>([
  {
    index: 0,
    id: 21,
    name: 'Red Effect 1',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+1',
  },
  {
    index: 1,
    id: 22,
    name: 'Red Effect 2',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+2',
  },
  {
    index: 2,
    id: 23,
    name: 'Red Effect 3',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+3',
  },
  {
    index: 3,
    id: 24,
    name: 'Red Effect 4',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+4',
  },
  {
    index: 4,
    id: 25,
    name: 'Red Effect 5',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+5',
  },
  {
    index: 5,
    id: 26,
    name: 'Red Effect 6',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+6',
  },
  {
    index: 6,
    id: 27,
    name: 'Red Effect 7',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+7',
  },
  {
    index: 7,
    id: 28,
    name: 'Red Effect 8',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+8',
  },
  {
    index: 8,
    id: 29,
    name: 'Red Effect 9',
    imageUrl: 'https://via.placeholder.com/200x150/ff2236/ffffff?text=Red+9',
  },
])

const signalButtons = [
  { label: 'BLUE', id: 9, color: '#0000ff' },
  { label: 'YELLOW', id: 10, color: '#FFEB3B' },
  { label: 'RED', id: 11, color: '#FF2236' },
]

const selectedEffect = ref<null | Effect>(null)
const selectedSignalId = ref<number | null>(null)
const gain = ref<number>(0.5)
const signalDuration = ref<number>(0.5)

// 選択されている SignalButton の色を取得
const selectedSignalColor = computed(() => {
  if (selectedSignalId.value === null) return '#bbbbbb' // SignalButtonが選択されていない場合は白
  const selectedButton = signalButtons.find((btn) => btn.id === selectedSignalId.value)
  return selectedButton?.color ?? '#fff'
})

// 選択されている色に応じたエフェクトリストを返す
const currentEffectList = computed(() => {
  // 色が選択されていない場合はBLUEをデフォルト表示（グレーアウト状態）
  if (selectedSignalId.value === null) return blueEffectList.value

  const selectedButton = signalButtons.find((btn) => btn.id === selectedSignalId.value)
  if (!selectedButton) return blueEffectList.value

  switch (selectedButton.label) {
    case 'BLUE':
      return blueEffectList.value
    case 'YELLOW':
      return yellowEffectList.value
    case 'RED':
      return redEffectList.value
    default:
      return blueEffectList.value
  }
})

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
let emissionCounter = 0 // Signal/Effect/Effect/Effect パターンのカウンター

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

// 一元化された信号送信関数
function startSignalEmission() {
  if (!isHostActive || !audioContext) return

  // 既存のインターバルをクリア
  stopAudio()

  // カウンターをリセット
  emissionCounter = 0

  // どちらも選択されていない場合は停止
  if (!selectedEffect.value && selectedSignalId.value === null) {
    return
  }

  // Effectのみ選択されている場合
  if (selectedEffect.value && selectedSignalId.value === null) {
    sendEffect(selectedEffect.value.id, signalDuration.value)
    return
  }

  // Signalのみ選択されている場合
  if (!selectedEffect.value && selectedSignalId.value !== null) {
    sendEffect(selectedSignalId.value, signalDuration.value)
    return
  }

  // 両方選択されている場合：Signal/Effect/Effect/Effect パターン
  if (selectedEffect.value && selectedSignalId.value !== null) {
    // 最初の送信を即座に実行
    sendSignalOrEffect()

    // インターバルを設定
    interval = setInterval(
      () => {
        sendSignalOrEffect()
      },
      signalDuration.value * 1000 * 1.5,
    )
  }
}

// Signal/Effect/Effect/Effect パターンで送信
function sendSignalOrEffect() {
  if (!selectedEffect.value || selectedSignalId.value === null) return

  // カウンターに基づいて送信するIDを決定
  // 0: Signal, 1-3: Effect
  const shouldSendSignal = emissionCounter % 4 === 0
  const idToSend = shouldSendSignal ? selectedSignalId.value : selectedEffect.value.id

  // 実際の送信（sendEffect関数を使わずに直接実装）
  sendSingleSignal(idToSend, signalDuration.value)

  // カウンターをインクリメント
  emissionCounter++
}

// 単一の信号を送信する関数（sendEffectから抽出）
async function sendSingleSignal(id: number, duration: number) {
  if (!isHostActive || !audioContext) return

  const frequency = getFrequencyForChannel(id)
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

  await delay(10)

  hostOscillator = audioContext.createOscillator()
  hostOscillator.type = 'sine'
  hostOscillator.frequency.setValueAtTime(frequency, audioContext.currentTime)
  hostOscillator.connect(hostGain!)

  // エンベロープを設定
  now = audioContext.currentTime
  hostGain!.gain.setValueAtTime(0, now)
  hostGain!.gain.linearRampToValueAtTime(gain.value, now + 0.01)
  hostGain!.gain.setValueAtTime(gain.value, now + duration - 0.01)
  hostGain!.gain.linearRampToValueAtTime(0, now + duration)

  // 信号を送信
  hostOscillator.start(now)
  hostOscillator.stop(now + duration)

  console.log(`信号送信: ID=${id}, 周波数=${frequency}Hz`)
}

function stopEffect() {
  if (!isHostActive || !audioContext) return

  // 選択状態をクリア
  selectedEffect.value = null
  selectedSignalId.value = null

  // 音声を停止
  stopAudio()
}

// 音声のみを停止する関数（選択状態は維持）
function stopAudio() {
  if (!audioContext) return

  const now = audioContext.currentTime

  if (hostOscillator) {
    hostGain!.gain.cancelScheduledValues(now)
    hostGain!.gain.setValueAtTime(hostGain!.gain.value, now)
    hostGain!.gain.linearRampToValueAtTime(0, now + 0.01)
    hostOscillator.stop(now + 0.01)
  } else {
    if (hostGain) {
      hostGain.gain.cancelScheduledValues(now)
      hostGain.gain.setValueAtTime(0, now)
    }
  }

  if (interval) clearInterval(interval)
  emissionCounter = 0
}

onMounted(() => {
  initializeHost()
})

function onSelectorButtonClick(effect: Effect) {
  // 同じエフェクトがクリックされた場合は選択を解除
  if (selectedEffect.value?.id === effect.id) {
    selectedEffect.value = null
  } else {
    selectedEffect.value = effect
  }
  // SignalButtonの選択は維持する
  startSignalEmission()
}

function onSignalButtonClick(id: number) {
  // 同じシグナルがクリックされた場合は選択を解除
  if (selectedSignalId.value === id) {
    selectedSignalId.value = null
    selectedEffect.value = null
  } else {
    // 異なる色に切り替えた場合は、エフェクトの選択をクリア
    if (selectedSignalId.value !== null && selectedSignalId.value !== id) {
      selectedEffect.value = null
    }
    selectedSignalId.value = id
  }
  startSignalEmission()
}

function resetGain() {
  gain.value = 0.5
}

function resetDuration() {
  signalDuration.value = 0.5
}

// 現在選択されているエフェクトの表示テキストを生成
const nowPlayingText = computed(() => {
  let id = 0
  let name = 'NONE'

  // 両方選択されている場合
  if (selectedEffect.value && selectedSignalId.value !== null) {
    const signal = signalButtons.find((s) => s.id === selectedSignalId.value)
    const signalName = signal?.label || 'SIGNAL'
    const effectName = selectedEffect.value.name
    return `[SIGNAL: ${signalName}] + [EFFECT: ${effectName}] - ID: ${selectedSignalId.value}/${selectedEffect.value.id} - `
  }

  // Effectのみ選択されている場合
  if (selectedEffect.value) {
    id = selectedEffect.value.id
    name = selectedEffect.value.name
  } else if (selectedSignalId.value !== null) {
    // Signalのみ選択されている場合
    const signal = signalButtons.find((s) => s.id === selectedSignalId.value)
    if (signal) {
      id = signal.id
      name = signal.label
    }
  } else {
    return 'NOW PLAYING: NO EFFECT SELECTED / ODOL SIGNAL / '
  }

  return `NOW PLAYING: [${id}] ${name} / ODOL SIGNAL / `
})
</script>

<template>
  <div class="app-container">
    <BackgroundCanvas :selectedColor="selectedSignalColor" />
    <div class="headline-container">
      <div class="headline-scroll">
        <span class="headline-text">{{ nowPlayingText.repeat(10) }}</span>
      </div>
    </div>
    <!-- Effects Grid - Full Width -->
    <div class="effects-grid">
      <SelectorButton
        v-for="effect in currentEffectList"
        :key="effect.id"
        :id="effect.id"
        :index="effect.index"
        :name="effect.name"
        :imageUrl="effect.imageUrl"
        :isSelected="selectedEffect?.index === effect.index"
        :borderColor="selectedSignalColor"
        :disabled="selectedSignalId === null"
        @select="onSelectorButtonClick(effect)"
      >
      </SelectorButton>
    </div>

    <!-- Control Panel - Below Effects Grid -->
    <div class="control-panel">
      <!-- Left Side - Color Selector -->
      <div class="left-controls">
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
      </div>

      <!-- Right Side - Sliders and Stop Button -->
      <div class="right-controls">
        <!-- Sliders -->
        <div class="sliders-container">
          <div class="slider-group">
            <div class="slider-header">
              <span class="slider-label">GAIN {{ gain }}</span>
              <button class="reset-button" @click="resetGain">
                <IconRefresh />
              </button>
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
              <button class="reset-button" @click="resetDuration">
                <IconRefresh />
              </button>
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
  font-family: 'Roboto Mono', monospace;
}
</style>

<style scoped>
/* Dark theme base */
.app-container {
  background-color: transparent;
  color: #fff;
  width: 100vw;
  height: 100vh;
  font-family: 'Roboto Mono', monospace;
  margin: 0;
  padding: 0;
  overflow: hidden;
  position: relative;
}

/* Headline Scrolling Container */
.headline-container {
  width: 100vw;
  overflow: hidden;
  padding: 20px 0;
  position: relative;
}

.headline-scroll {
  display: inline-block;
  white-space: nowrap;
  animation: scroll-left 20s linear infinite;
}

.headline-text {
  color: #00c19c;
  font-size: 3rem;
  font-weight: bold;
  display: inline-block;
}

@keyframes scroll-left {
  0% {
    transform: translateX(0);
  }
  100% {
    transform: translateX(-10%);
  }
}

/* Effects Grid - Full Width */
.effects-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 10px;
  height: 55vh;
  padding: 20px;
  margin-bottom: 30px;
}

/* Control Panel - Below Effects Grid */
.control-panel {
  padding: 20px;
  display: flex;
  flex-direction: row;
  gap: 40px;
  align-items: center;
  justify-content: space-between;
  height: 35vh;
  box-sizing: border-box;
}

/* Left Controls */
.left-controls {
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Right Controls */
.right-controls {
  max-width: 32vw;
  display: flex;
  flex-direction: row;
  gap: 20px;
  align-items: center;
  flex: 1;
}

/* Color Selector */
.color-selector {
  display: flex;
  flex-direction: row;
  gap: 20px;
  background-color: transparent;
  padding: 0;
  justify-content: center;
}

/* Sliders */
.sliders-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
  flex: 1;
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
  width: 32px;
  height: 32px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  padding: 0;
}

.reset-button svg {
  width: 20px;
  height: 20px;
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
  display: flex;
  justify-content: center;
  align-items: center;
  flex-shrink: 0;
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
