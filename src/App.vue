<script setup lang="ts">
import { ref, onMounted } from 'vue';
import SelectorButton from './components/SelectorButton.vue';
import SelectorButtonEditor from './components/SelectorButtonEditor.vue';
import MiniMap from './components/MiniMap.vue';

type Effect = {
  id: number;
  name: string;
};
const webSocketUrl = 'ws://localhost:8080';
let webSocket: WebSocket | null = null;

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

// 音声送信

const CONFIG = {
            BASE_FREQ: 17000,      // 基本周波数
            FREQ_RANGE: 2000,      // 周波数範囲
            NUM_CHANNELS: 32,      // チャンネル数
            SIGNAL_DURATION: 1.0,  // 信号長（秒）
            SAMPLE_RATE: 44100,    // サンプリングレート
            FFT_SIZE: 8192,        // FFT サイズ
            DETECTION_THRESHOLD: 0.3 // 検出閾値
        };
        
        // グローバル変数
let audioContext : any= null;
let hostOscillator : any= null;
let hostGain : any = null;
let isHostActive = false;
let signalFn : any = null;

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
  return new Promise( (resolve) =>  {
    setTimeout(() => {
      resolve(true);
    }, ms)
  })
}       

        // エフェクト送信
 async function sendEffect(effectId: number) {
            if (!isHostActive || !audioContext) return;
            
            const frequency = getFrequencyForChannel(effectId);
            
            let now = audioContext.currentTime;
            hostGain.gain.linearRampToValueAtTime(0, now + 0.5);

            // 既存のオシレーターを停止
            
            let wait = await delay(500);
            
            if(wait) {

              if (hostOscillator) {
                hostOscillator.stop();
              }

              let retfn = () => {
                
                hostOscillator = audioContext.createOscillator();
                hostOscillator.type = 'sine';
                hostOscillator.frequency.setValueAtTime(frequency, audioContext.currentTime);
                hostOscillator.connect(hostGain);
                // エンベロープを設定（フェードイン・フェードアウト）
                now = audioContext.currentTime;
                hostGain.gain.cancelScheduledValues(now);
                hostGain.gain.setValueAtTime(0, now);
                hostGain.gain.linearRampToValueAtTime(gain.value, now + 0.1);
                // hostGain.gain.setValueAtTime(0.1, now + CONFIG.SIGNAL_DURATION - 0.1);
                hostGain.gain.linearRampToValueAtTime(0, now + CONFIG.SIGNAL_DURATION);
                
                // 信号を送信
                hostOscillator.start(now);
                hostOscillator.stop(now + CONFIG.SIGNAL_DURATION);
                
                // UI更新
                
                console.log(`エフェクト送信: ID=${effectId}, 周波数=${frequency}Hz`);
              }
  
              signalFn = retfn;
            }
            // 新しいオシレーターを作成
        }

onMounted(() => {
  initializeHost();
  connectWebSocket(); 
  if(localStorage.getItem('effectList')) {
    effectList.value = JSON.parse(localStorage.getItem('effectList') || "[]");
  } 
  else {
    localStorage.setItem('effectList', JSON.stringify(effectList.value));
  }
})

function connectWebSocket() {
  if (webSocket && webSocket.readyState === WebSocket.OPEN) {
    return; // Already connected
  }
  webSocket = new WebSocket(webSocketUrl);
  webSocket.onopen = () => {
    connectionStatus.value = 'Connected';
  };

  webSocket.onclose = () => {
    connectionStatus.value = 'Disconnected';
  };

  webSocket.onerror = (error) => {
    console.error('WebSocket Error:', error);
    connectionStatus.value = 'Error';
  };
}

setInterval(() => {
  if(typeof signalFn === 'function') signalFn();
}, CONFIG.SIGNAL_DURATION * 1000 * 2);

function onSelectorButtonClick(effect: Effect) {
  selectedEffect.value = effect;
  if (audioContext) {
    signalFn = sendEffect(effect.id);
  }
  if (!webSocket) return;
  if (effect && webSocket.readyState === WebSocket.OPEN) {
    webSocket.send(JSON.stringify({ action: 'selectEffect', effectId: effect.id }));
  }
}

function onSelectorEdit( props: Effect, index: number ){
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
        <div class="websocket-status">
          <p>Web Socket 接続状況: {{ connectionStatus }}</p>
          <button @click="connectWebSocket">再接続</button>
        </div>
        <div class="websocket-status">
          <p>サウンドエンコーダーの状態： {{ encoderStatus }}</p>
          <input id="gain" type="range" step="0.1" min="0" max="1" v-model="gain">
          <label for="gain">Gain</label>
          <p> {{ gain }} </p>
        </div>
        <div class="websocket-status">
          <p>エフェクト名を編集する</p>
          <button @click="isEditing = !isEditing">編集</button>
        </div>
        <p>選択中のエフェクト: {{ selectedEffect ? selectedEffect.name : 'なし' }}</p>
      </div>
      <div>
        <MiniMap 
          :effectList="effectList" 
          :selectedEffect="selectedEffect"
          :page="page"> 
        </MiniMap>
      </div>
    </div>
    <div>
      <div v-if="!isEditing" class="selector">
        <SelectorButton 
          v-for="effect in effectList.slice((page - 1) * 16, page * 16)" :key="effect.id" 
          :id="effect.id" 
          :name="effect.name"
          :isSelected="selectedEffect?.id === effect.id"
          @select="onSelectorButtonClick(effect)" 
          >
        </SelectorButton>
      </div>
      <div v-else class="selector">
        <SelectorButtonEditor
          v-for="effect in effectList.slice((page - 1) * 16, page * 16)" :key="effect.id" 
          :id="effect.id" 
          :name="effect.name"
          @edit="e => onSelectorEdit(e, effect.id)"
          >
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
.selector {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin: 20px 0;
}

.websocket-status {
  display: flex;
  flex-direction: row;
  gap: 10px;
  align-items: center;
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
