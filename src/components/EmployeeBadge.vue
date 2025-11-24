<template>
  <div class="canvas-container">
    <!-- 54mm x 85.5mm at 300 DPI = 638px x 1012px -->
    <canvas ref="badgeCanvas" width="638" height="1012"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { addMetadata, addMetadataFromBase64DataURI } from 'meta-png'
import MainSvg from '../assets/Main.svg'

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
let svgBgImage = null // 固定的 SVG 背景
let bgImage = null
let photoImg = null

onMounted(() => {
  ctx = badgeCanvas.value.getContext('2d')
  loadSvgBackground()
  loadBackgroundImage()
  loadPhotoImage()
})

// 載入固定的 SVG 背景
const loadSvgBackground = () => {
  svgBgImage = new Image()
  svgBgImage.onload = () => {
    drawBadge()
  }
  svgBgImage.onerror = () => {
    console.error('SVG 背景載入失敗')
    svgBgImage = null
    drawBadge()
  }
  svgBgImage.src = MainSvg
}

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

  const width = 638
  const height = 1012

  // 清空畫布
  ctx.clearRect(0, 0, width, height)

  // 1. 先繪製固定的 SVG 背景（最底層）
  if (svgBgImage && svgBgImage.complete) {
    ctx.drawImage(svgBgImage, 0, 0, width, height)
  } else {
    // 如果 SVG 未載入，顯示白色背景
    ctx.fillStyle = '#ffffff'
    ctx.fillRect(0, 0, width, height)
  }

  // 2. 如果有額外的背景圖片，可以疊加在 SVG 上（可選）
  // 這裡可以保留或移除，視需求而定
  
  // 3. 照片區域（根據 SVG 的預留位置）
  const photoX = 177
  const photoY = 217
  const photoWidth = 380
  const photoHeight = 456
  
  if (photoImg && photoImg.complete) {
    // 計算照片的縮放比例以填滿區域（保持比例並裁切）
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
  }

  // 4. 姓名區域（酒紅色區塊內，白色文字）
  const nameBlockY = 727
  const nameBlockHeight = 87
  const nameCenterY = nameBlockY + nameBlockHeight / 2 + 3 // 垂直置中並微調
  
  ctx.fillStyle = '#ffffff'
  ctx.font = 'bold 48px Arial'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText(props.employeeName || '未輸入', 367, nameCenterY)

  // 5. 員工編號區域（酒紅色區塊內，白色文字）
  const idBlockY = 868
  const idBlockHeight = 87
  const idCenterY = idBlockY + idBlockHeight / 2 + 3 // 垂直置中並微調
  
  ctx.fillStyle = '#ffffff'
  ctx.font = 'bold 48px Arial'
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText(props.employeeId || '未輸入', 367, idCenterY)
}

defineExpose({
  downloadBadge: async (callback) => {
    if (!badgeCanvas.value) return
    
    try {
      // 將 canvas 轉換為 Data URL
      const dataUrl = badgeCanvas.value.toDataURL('image/png')
      
      // 準備元數據
      const metadata = {
        'Title': '員工證',
        'Author': '公司名稱',
        'EmployeeName': props.employeeName || '未輸入',
        'EmployeeId': props.employeeId || '未輸入',
        'Software': 'Employee Badge Generator',
        'CreationTime': new Date().toISOString(),
        'DPI': '300'
      }
      
      // 使用 meta-png 的 addMetadataFromBase64DataURI 逐一加入元數據
      let resultDataUrl = dataUrl
      for (const [key, value] of Object.entries(metadata)) {
        resultDataUrl = addMetadataFromBase64DataURI(resultDataUrl, key, value)
      }
      
      // 統一使用 callback 傳遞圖片資料給 dialog
      if (callback) {
        callback(resultDataUrl)
      } else {
        // 如果沒有 callback（向下相容），使用標準下載方式
        const link = document.createElement('a')
        link.download = `員工證_${props.employeeName || 'unnamed'}_${props.employeeId || 'noid'}.png`
        link.href = resultDataUrl
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)
      }
    } catch (error) {
      console.error('下載失敗:', error)
      alert('下載失敗,請稍後再試')
    }
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
