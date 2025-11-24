<template>
  <div class="canvas-container">
    <canvas ref="badgeCanvas" width="400" height="600"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const props = defineProps({
  employeeName: {
    type: String,
    default: ''
  },
  employeeId: {
    type: String,
    default: ''
  },
  backgroundImage: {
    type: String,
    default: ''
  },
  photoImage: {
    type: String,
    default: ''
  }
})

const badgeCanvas = ref(null)
let ctx = null
let bgImage = null
let photoImg = null

onMounted(() => {
  ctx = badgeCanvas.value.getContext('2d')
  loadBackgroundImage()
  loadPhotoImage()
})

watch(() => [props.employeeName, props.employeeId], () => {
  drawBadge()
})

watch(() => props.backgroundImage, () => {
  loadBackgroundImage()
})

watch(() => props.photoImage, () => {
  loadPhotoImage()
})

const loadBackgroundImage = () => {
  if (props.backgroundImage) {
    bgImage = new Image()
    bgImage.crossOrigin = 'anonymous' // 避免跨域問題
    bgImage.onload = () => {
      drawBadge()
    }
    bgImage.onerror = () => {
      console.error('背景圖片載入失敗')
      bgImage = null
      drawBadge()
    }
    bgImage.src = props.backgroundImage
  } else {
    bgImage = null
    drawBadge()
  }
}

const loadPhotoImage = () => {
  if (props.photoImage) {
    photoImg = new Image()
    photoImg.crossOrigin = 'anonymous'
    photoImg.onload = () => {
      drawBadge()
    }
    photoImg.onerror = () => {
      console.error('照片載入失敗')
      photoImg = null
      drawBadge()
    }
    photoImg.src = props.photoImage
  } else {
    photoImg = null
    drawBadge()
  }
}

const drawBadge = () => {
  if (!ctx) return

  // 清空畫布
  ctx.clearRect(0, 0, 400, 600)

  // 背景圖片或純色背景
  if (bgImage && bgImage.complete) {
    // 繪製背景圖片，自動縮放以填滿整個 canvas
    ctx.drawImage(bgImage, 0, 0, 400, 600)
  } else {
    // 預設白色背景
    ctx.fillStyle = '#ffffff'
    ctx.fillRect(0, 0, 400, 600)
  }

  // 頂部藍色區塊（可以選擇是否保留，或調整透明度）
  ctx.fillStyle = 'rgba(37, 99, 235, 0.9)'
  ctx.fillRect(0, 0, 400, 150)

  // 公司名稱
  ctx.fillStyle = '#ffffff'
  ctx.font = 'bold 28px Arial'
  ctx.textAlign = 'center'
  ctx.fillText('公司名稱', 200, 60)

  ctx.font = '18px Arial'
  ctx.fillText('EMPLOYEE BADGE', 200, 90)

  // 邊框
  ctx.strokeStyle = '#2563eb'
  ctx.lineWidth = 3
  ctx.strokeRect(10, 10, 380, 580)

  // 照片區域
  if (photoImg && photoImg.complete) {
    // 計算照片的縮放比例以填滿區域（保持比例並裁切）
    const photoWidth = 200
    const photoHeight = 240
    const photoX = 100
    const photoY = 180
    
    const scale = Math.max(photoWidth / photoImg.width, photoHeight / photoImg.height)
    const scaledWidth = photoImg.width * scale
    const scaledHeight = photoImg.height * scale
    const offsetX = (scaledWidth - photoWidth) / 2
    const offsetY = (scaledHeight - photoHeight) / 2
    
    // 裁切照片以填滿區域
    ctx.save()
    ctx.beginPath()
    ctx.rect(photoX, photoY, photoWidth, photoHeight)
    ctx.clip()
    ctx.drawImage(photoImg, photoX - offsetX, photoY - offsetY, scaledWidth, scaledHeight)
    ctx.restore()
  } else {
    // 照片佔位符
    ctx.fillStyle = '#e5e7eb'
    ctx.fillRect(100, 180, 200, 240)
    
    ctx.fillStyle = '#9ca3af'
    ctx.font = '16px Arial'
    ctx.fillText('照片區域', 200, 300)
  }

  // 姓名標籤
  ctx.fillStyle = '#374151'
  ctx.font = 'bold 20px Arial'
  ctx.textAlign = 'center'
  ctx.fillText('姓名', 200, 460)

  // 姓名內容
  ctx.fillStyle = '#1f2937'
  ctx.font = 'bold 26px Arial'
  ctx.fillText(props.employeeName || '未輸入', 200, 495)

  // 員工編號標籤
  ctx.fillStyle = '#374151'
  ctx.font = 'bold 20px Arial'
  ctx.fillText('員工編號', 200, 535)

  // 員工編號內容
  ctx.fillStyle = '#1f2937'
  ctx.font = 'bold 24px Arial'
  ctx.fillText(props.employeeId || '未輸入', 200, 570)
}

defineExpose({
  downloadBadge: () => {
    if (!badgeCanvas.value) return
    
    const link = document.createElement('a')
    link.download = `員工證_${props.employeeName || 'unnamed'}_${props.employeeId || 'noid'}.png`
    link.href = badgeCanvas.value.toDataURL()
    link.click()
  }
})
</script>

<style scoped>
.canvas-container {
  display: flex;
  justify-content: center;
  padding: 20px;
  background: #f9fafb;
  border-radius: 12px;
}

canvas {
  border: 1px solid #d1d5db;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
</style>
