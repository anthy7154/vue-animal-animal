<script setup>
import { ref } from 'vue'
import { onBeforeUnmount, computed, onMounted, onUnmounted, watch } from 'vue'

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
  run()
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
const currentIndex = ref(0)
const intervalId = ref(0)
const run = () => {
  intervalId.value = setInterval(() => {
    currentIndex.value < 8 ? currentIndex.value++ : currentIndex.value = 0;
  }, 500);
}

// 倒數讀秒
const showCountDown = ref(0)

// 監聽 currentIndex 變化，更新倒數讀秒，並在到達 8 時重置動畫
watch(currentIndex, () => {
  if (currentIndex.value === 8) {
    stop()
    randomIndex()
    run()
  }
  showCountDown.value = 8 - currentIndex.value
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

// // 自動倒數計時控制 ==============================
// const countDown = ref(0)

// // 重置倒數讀秒
// const refreshCountDown = () => {
//   stop()
//   showCountDown.value = countDown.value
//   run()
// }

</script>

<template>
  <div class="container-fluid">
    <div class="row">
      <div class="col-12">
        <h1 class="h1" v-if="showCountDown">{{ showCountDown }}</h1>
        <h1 class="h1" v-else>準備開始遊戲囉！</h1>
        <input ref="uploadFileRef" type="file" name="imgUpload" multiple="multiple"
          @change="changeFile($event)" />
      </div>
    </div>

    <div class="row" v-if="imageIndex.length > 0">
      <template v-for="(i, index) in imageIndex" :key="index">
        <div class="col-3" :class="`${currentIndex === index ? 'highlight' : 'show'}`">
          <img class="image-item" :id="`image-${index}`" :src="previews[i]?.url" :alt="previews[i]?.name" />
          <p>{{ index }}</p>
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
</style>
