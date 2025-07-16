<script setup lang="ts">
import { ref, onMounted } from 'vue';
import SelectorButton from './components/SelectorButton.vue';
import SelectorButtonEditor from './components/SelectorButtonEditor.vue';
import MiniMap from './components/MiniMap.vue';
import StopButton from './components/StopButton.vue';

type Effect = {
  id: number;
  name: string;
};

const effectLength = 32;
const effectList = ref<any>(Array.from({ length: effectLength }, (_, i) => ({
  id: i,
  name: `Effect ${i + 1}`
})));

const page = ref(1);
const pageLength = effectLength / 16; // Assuming 16 effects per page
const connectionStatus = ref('Unconnected');
const encoderStatus = ref('standby')
const selectedEffect = ref<null | Effect>(null);
const isEditing = ref<boolean>(false);
const gain = ref<number>(0.1);
const signalDuration = ref<number>(0.5);

// 音声送信

const CONFIG = {
  BASE_FREQ: 17000,      // 基本周波数
  FREQ_RANGE: 2000,      // 周波数範囲
  NUM_CHANNELS: 32,      // チャンネル数
  // SIGNAL_DURATION: 1.0,  // 信号長（秒）
  SAMPLE_RATE: 44100,    // サンプリングレート
  FFT_SIZE: 8192,        // FFT サイズ
  DETECTION_THRESHOLD: 0.3 // 検出閾値
};

// グローバル変数
let audioContext: any = null;
let hostOscillator: any = null;
let hostGain: any = null;
let isHostActive = false;
let interval: any = null;

function getFrequencyForChannel(channel: number) {
  const step = CONFIG.FREQ_RANGE / CONFIG.NUM_CHANNELS;
  return CONFIG.BASE_FREQ + (channel * step);
}

async function initializeHost() {
  try {
    audioContext = new (window.AudioContext)();

    // ゲインノードを作成
    hostGain = audioContext.createGain();
    hostGain.connect(audioContext.destination);
    hostGain.gain.setValueAtTime(0, audioContext.currentTime);

    isHostActive = true;
    encoderStatus.value = "ホスト初期化に成功しました"

  } catch (error) {
    console.error('ホスト初期化エラー:', error);
    encoderStatus.value = 'ホスト初期化に失敗しました';
  }
}

function delay(ms: number): Promise<boolean> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(true);
    }, ms)
  })
}

// エフェクト送信
async function sendEffect(effectId: number, duration: number) {
  if (!isHostActive || !audioContext) return;

  const frequency = getFrequencyForChannel(effectId);

  let now = audioContext.currentTime;

  if (hostOscillator) {
    hostGain.gain.cancelScheduledValues(now);
    hostGain.gain.setValueAtTime(hostGain.gain.value, now);
    hostGain.gain.linearRampToValueAtTime(0, now + 0.01);
    hostOscillator.stop(now + 0.01);
  } else {
    hostGain.gain.cancelScheduledValues(now);
    hostGain.gain.setValueAtTime(0, now);
  }
  // hostGain.gain.linearRampToValueAtTime(0, now + 0.1);

  // 既存のオシレーターを停止

  let wait = await delay(10);

  if (wait) {
    if (interval) clearInterval(interval);
    let retfn = () => {

      hostOscillator = audioContext.createOscillator();
      hostOscillator.type = 'sine';
      hostOscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);
      hostOscillator.connect(hostGain);
      // エンベロープを設定（フェードイン・フェードアウト）
      now = audioContext.currentTime;
      hostGain.gain.setValueAtTime(0, now);
      hostGain.gain.linearRampToValueAtTime(gain.value, now + 0.01);
      hostGain.gain.setValueAtTime(gain.value, now + duration - 0.01);
      hostGain.gain.linearRampToValueAtTime(0, now + duration);

      // 信号を送信
      hostOscillator.start(now);
      hostOscillator.stop(now + duration);

      // UI更新

      console.log(`エフェクト送信: ID=${effectId}, 周波数=${frequency}Hz`);
    }

    interval = setInterval(() => {
      if (typeof retfn === 'function') retfn();
    }, (duration * 1000) * 1.5);
  }
  // 新しいオシレーターを作成
}

function stopEffect() {
  if (!isHostActive || !audioContext) return;

  selectedEffect.value = null;

  let now = audioContext.currentTime;

  if (hostOscillator) {
    hostGain.gain.cancelScheduledValues(now);
    hostGain.gain.setValueAtTime(hostGain.gain.value, now);
    hostGain.gain.linearRampToValueAtTime(0, now + 0.01);
    hostOscillator.stop(now + 0.01);
  } else {
    hostGain.gain.cancelScheduledValues(now);
    hostGain.gain.setValueAtTime(0, now);
  }

  clearInterval(interval);
}

onMounted(() => {
  initializeHost();
  if (localStorage.getItem('effectList')) {
    effectList.value = JSON.parse(localStorage.getItem('effectList') || "[]");
  }
  else {
    localStorage.setItem('effectList', JSON.stringify(effectList.value));
  }
})



function onSelectorButtonClick(effect: Effect) {
  selectedEffect.value = effect;
  if (audioContext) {
    sendEffect(effect.id, signalDuration.value);
  }
}

function onSelectorEdit(props: Effect, index: number) {
  effectList.value[index] = props;
  localStorage.setItem('effectList', JSON.stringify(effectList.value));
  console.log(`Effect ${index} updated:`, props);
}

function nextPage() {
  if (page.value < pageLength) {
    page.value++;
  }
}

function previousPage() {
  if (page.value > 1) {
    page.value--;
  }
}

</script>

<template>
  <div class="app-container">
    <div>
      <div>
        <h1>ASP Sync Effect Controller</h1>
        <div class="flex-row gap-20">
          <div>
            <div class="flex-column">
              <p>サウンドエンコーダーの状態： {{ encoderStatus }}</p>
              <div class="flex-column">
                <div class="flex-row">
                  <input id="gain" type="range" step="0.1" min="0" max="1" v-model.number="gain">
                  <label for="gain">gain</label>
                  <p> {{ gain }} </p>
                </div>
                <div class="flex-row">
                  <input id="duration" type="range" step="0.1" min="0.2" max="1" v-model.number="signalDuration">
                  <label for="duration">duration</label>
                  <p> {{ signalDuration }}</p>
                </div>
              </div>
              <div class="flex-row">
                <p>エフェクト名を編集する</p>
                <button @click="isEditing = !isEditing">編集</button>
              </div>
              <p>選択中のエフェクト: {{ selectedEffect ? selectedEffect.name : 'なし' }}</p>
            </div>
          </div>
          <div>
            <StopButton :isSelected="!selectedEffect" @stop="stopEffect()"></StopButton>
          </div>
        </div>
      </div>
      <div>
        <MiniMap :effectList="effectList" :selectedEffect="selectedEffect" :page="page">
        </MiniMap>
      </div>
    </div>
    <div>
      <div v-if="!isEditing" class="selector">
        <SelectorButton v-for="effect in effectList.slice((page - 1) * 16, page * 16)" :key="effect.id" :id="effect.id"
          :name="effect.name" :isSelected="selectedEffect?.id === effect.id" @select="onSelectorButtonClick(effect)">
        </SelectorButton>
      </div>
      <div v-else class="selector">
        <SelectorButtonEditor v-for="effect in effectList.slice((page - 1) * 16, page * 16)" :key="effect.id"
          :id="effect.id" :name="effect.name" @edit="e => onSelectorEdit(e, effect.id)">
        </SelectorButtonEditor>
      </div>
    </div>
    <div class="pagenation">
      <button @click="previousPage()">前のページ</button>
      <p>{{ page }} / {{ pageLength }}</p>
      <button @click="nextPage()">次のページ</button>
    </div>
  </div>
</template>

<style scoped>
p {
  margin: 5px 0px;
}

.selector {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin: 20px 0;
}

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

.gap-20 {
  gap: 6rem;
}

.app-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.pagenation {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  gap: 10px;
}
</style>
