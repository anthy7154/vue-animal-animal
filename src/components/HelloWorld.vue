<script setup>
import { ref, onBeforeUnmount, onUnmounted, watch, onMounted } from 'vue'
import animalMusic from '../assets/animal-music.mp3'

defineProps({
  msg: String,
})


// 上傳圖片
const previews = ref([])
const uploadFileRef = ref(null)

function changeFile(e) {
  const files = e.target.files
  if (!files || files.length === 0) return

  // revoke old object URLs to avoid memory leaks
  previews.value.forEach((p) => URL.revokeObjectURL(p.url))
  previews.value = []

  Array.from(files).forEach((file) => {
    const item = {
      name: file.name,
      url: URL.createObjectURL(file),
    }
    previews.value.push(item)
  })

  // Clear input value so change event fires again when the same file is re-selected
  if (e.target) e.target.value = ''

  // 產生隨機數字陣列並開始跑動畫
  randomIndex()
}


// 處理上傳圖片後畫面重整
onBeforeUnmount(() => {
  previews.value.forEach((p) => URL.revokeObjectURL(p.url))
})

// ============
// 產生隨機數字陣列
const max = ref(0)
const min = ref(0)
const r = ref(0)
const imageIndex = ref([])

const randomIndex = () => {
  max.value = previews.value.length
  if (!max) {
    imageIndex.value = []
    return
  }
  imageIndex.value.splice(0, imageIndex.value.length)

  for (let i = 0; i < 8; i++) {
    r.value = Math.floor(Math.random() * (max.value - min.value)) + min.value
    imageIndex.value.push(r.value)
  }
  // console.log('imageIndex:', imageIndex.value)
}

// ============
// highlight
const currentIndex = ref(-1)
const intervalId = ref(0)
const run = () => {
  if (gameStatus.value) {
    intervalId.value = setInterval(() => {
      currentIndex.value < 8 ? currentIndex.value++ : currentIndex.value = 0;
    }, 350);
  }
}

// 倒數讀秒
const showCountDown = ref(0)

// 監聽 currentIndex 變化，更新倒數讀秒，並在到達 8 時重置動畫
watch(currentIndex, () => {
  if (currentIndex.value === 8) {
    stop()
    if (previews.value.length) {
      randomIndex()
      setTimeout(() => {
        run()
      }, 1950);
    }
  }
  // showCountDown.value = 8 - currentIndex.value
})

// 結束倒數讀秒
const stop = () => {
  clearInterval(intervalId.value);
  intervalId.value = 0
}
// 清除計時器
onUnmounted(() => {
  clearInterval(intervalId.value)
  intervalId.value = 0
})

// ==============
const audioRef = ref(null);
const isPlaying = ref(false);
const currentTime = ref(0);
const duration = ref(0);
const currentAudioSrc = animalMusic

function togglePlay() {
  if (audioRef.value) {
    if (isPlaying.value) {
      audioRef.value.pause()
    } else {
      const p = audioRef.value.play()
      if (p && p.then) {
        p.then(() => { isPlaying.value = true }).catch((err) => { console.warn('play failed', err) })
      } else {
        isPlaying.value = true
      }
    }
  }
}

function updateTime() {
  currentTime.value = audioRef.value.currentTime;
}

function loadedMetadata() {
  duration.value = audioRef.value.duration;
}

onMounted(() => {
  // 初始狀態設定 (可選)
});

// ==============
const gameStatus = ref(false)
const gameStart = () => {
  // 檢查是否有上傳圖片
  if (previews.value.length) {
    gameStatus.value = !gameStatus.value
    togglePlay() // 開始播放音樂
    setTimeout(() => {
      stop()
      run()
    }, 4000);
  } else {
    alert('請先上傳圖片！')
  }
}

</script>

<template>
  <div class="container-fluid">
    <div class="row">

      <div class="col-12" v-if="imageIndex.length">
        <button class="btn btn-lg btn-success" @click="gameStart()">{{ isPlaying ? '結束遊戲' : '開始遊戲' }}</button>

        <audio ref="audioRef" :src="currentAudioSrc" @timeupdate="updateTime" @loadedmetadata="loadedMetadata"
          @play="isPlaying = true" @pause="isPlaying = false" @ended="isPlaying = false" preload="metadata"></audio>

        <!-- <span> -->
        <!-- <button class="btn btn-info" @click="togglePlay">{{ isPlaying ? 'II' : '>' }}</button> -->

        <!-- 進度條 -->
        <span class="process" v-if="currentAudioSrc">{{ Math.floor(Math.floor(currentTime) / Math.floor(duration) * 100)
        }}%</span>
        <!-- </span> -->
      </div>

      <div class="col-12">
        <!-- <h1 class="h1" v-if="showCountDown">倒數 {{ showCountDown }} 秒！</h1>
        <h1 class="h1" v-else>準備開始遊戲囉！</h1> -->

        <!-- 上傳圖片 -->
        <input v-if="!previews.length" ref="uploadFileRef" type="file" name="imgUpload" multiple="multiple"
          @change="changeFile($event)" />
      </div>
    </div>

    <div class="row show-area" v-if="imageIndex.length > 0 && gameStatus">
      <template v-for="(i, index) in imageIndex" :key="index">
        <div class="col-3" :class="`${currentIndex === index ? 'highlight' : 'show'}`">
          <img class="image-item" :id="`image-${index}`" :src="previews[i]?.url" :alt="previews[i]?.name" />
        </div>
      </template>
    </div>

  </div>
</template>

<style scoped>
.upload {
  position: fixed;
  top: 0px;
  left: 0;
  z-index: -100;
  opacity: 0;
}

.show-area {
  margin-top: 20px;
}

.show {
  border: 6px solid #000;
  width: 300px;
  height: 300px;
  overflow: hidden;
  margin: 0px 1px 1px 1px;
}

.image-item {
  width: 100%;
  height: 100%;
  object-fit: cover;
  /* 裁切填滿正方形區塊 */
  display: block;
}

.highlight {
  border: 20px solid #f00;
  width: 300px;
  height: 300px;
  overflow: hidden;
  margin: 0px 1px 1px 1px;
}

.process {
  margin-left: 20px;
  font-size: 1.2rem;
  font-weight: bold;
}
</style>
