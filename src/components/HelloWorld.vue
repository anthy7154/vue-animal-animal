<script setup>
import { ref, onBeforeUnmount, onUnmounted, watch, onMounted } from 'vue'
import animalMusic from '../assets/animal-music.mp3'

defineProps({
  msg: String,
})


// ==============
// 音樂播放控制
const audioRef = ref(null)
const isPlaying = ref(false)
const currentTime = ref(0)
const duration = ref(0)
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
  currentTime.value = audioRef.value.currentTime
}

function loadedMetadata() {
  duration.value = audioRef.value.duration
}

// ============
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
  imageIndex.value.splice(0, imageIndex.value.length)
  if (!max) {
    return
  }

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
const timeoutId = ref(0)
const run = () => {
  if (gameStatus.value) {
    intervalId.value = setInterval(() => {
      currentIndex.value < 8 ? currentIndex.value++ : currentIndex.value = 0
    }, 350)
  }
}

// 監聽 currentIndex 變化，更新倒數讀秒，並在到達 8 時重置動畫
watch(currentIndex, () => {
  if (currentIndex.value === 8 && gameStatus.value && isPlaying.value) {
    stop()
    if (previews.value.length) {
      randomIndex()
      timeoutId.value = setTimeout(() => {
        run()
      }, 2050)
    }
  }
})

// 結束倒數讀秒
const stop = () => {
  clearInterval(intervalId.value)
  intervalId.value = 0
  clearTimeout(timeoutId.value)
  timeoutId.value = 0
}

// 清除計時器
onUnmounted(() => {
  stop()
})

// ==============
const gameStatus = ref(false)
const gameStart = () => {
  stop()
  // 檢查是否有上傳圖片
  if (previews.value.length) {
    gameStatus.value = true
    togglePlay() // 開始播放音樂
    setTimeout(() => {
      run()
    }, 4000)
  } else {
    alert('請先上傳圖片！')
  }
}

const hardRefresh = () => {
  stop()
  audioRef.value.pause()
  audioRef.value.currentTime = 0
  window.location.reload()
}

const replay = () => {
  stop()
  audioRef.value.pause()
  audioRef.value.currentTime = 0
  gameStatus.value = false
  if (previews.value.length) {
    currentIndex.value = -1
    randomIndex()
  }
}

</script>

<template>
  <div class="container-fluid">
    <div class="row">

      <div class="col-12">
        <!-- music -->
        <audio ref="audioRef" :src="currentAudioSrc" @timeupdate="updateTime" @loadedmetadata="loadedMetadata"
          @play="isPlaying = true" @pause="isPlaying = false" @ended="isPlaying = false" preload="metadata"></audio>

        <!-- upload images -->
        <input v-if="!previews.length" ref="uploadFileRef" type="file" name="imgUpload" multiple="multiple"
          @change="changeFile($event)" />
      </div>

      <div class="col-12" id="option" v-if="imageIndex.length">

        <button class="btn btn-sm btn-warning" @click="hardRefresh">更換題目</button>
        <button class="btn btn-sm btn-danger" @click="replay" v-if="gameStatus">重新整理</button>
        <button class="btn btn-sm btn-success" @click="gameStart" v-if="!gameStatus">開始遊戲</button>

        <button class="btn btn-sm btn-outline-secondary process" disabled v-if="currentAudioSrc">進度 {{
          Math.floor(Math.floor(currentTime) / Math.floor(duration) *
            100)
        }}%</button>

        <div class="toast-container position-fixed top-50 start-50 translate-middle" v-if="currentTime >= duration">
          <div id="toastGameOver" class="toast show" role="alert" aria-live="assertive" aria-atomic="true">
            <div class="toast-body">GAME OVER!</div>
          </div>
        </div>
      </div>

    </div>

    <div class="row image-area" v-if="imageIndex.length > 0 && gameStart">
      <template v-for="(i, index) in imageIndex" :key="index">
        <div class="col-3" :class="`${currentIndex === index ? 'highlight' : 'normal'}`">
          <img class="image-item" :src="previews[i]?.url" :alt="previews[i]?.name" />
        </div>
      </template>
    </div>

  </div>
</template>

<style scoped>
#option .btn {
  margin: 0px 10px;
  color: #fff;
  font-size: 1.4rem;
  font-weight: bold;
}

.upload {
  position: fixed;
  top: 0px;
  left: 0;
  z-index: -100;
  opacity: 0;
}

.image-area {
  margin-top: 10px;
}

.normal {
  border: 6px solid #555;
  width: 350px;
  height: 350px;
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
  width: 350px;
  height: 350px;
  overflow: hidden;
  margin: 0px 1px 1px 1px;
}

.process {
  color: #000 !important;
}

.normal,
.highlight {
  padding-right: 0px !important;
  padding-left: 0px !important;
}

.toast-container,
.toast {
  pointer-events: none;
}

.toast-body {
  font-family: "Comic Sans MS";
  font-size: 5em;
  text-align: center;
  color: white;
  background-color: #ff0505;
  pointer-events: none;
}
</style>
