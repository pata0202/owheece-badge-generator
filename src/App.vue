<script setup>
import { ref } from 'vue'
import EmployeeBadge from './components/EmployeeBadge.vue'
import ImageCropper from './components/ImageCropper.vue'
import title from './assets/title.svg'

const employeeName = ref('')
const employeeId = ref('')
const backgroundImage = ref('')
const photoImage = ref('')
const badgeRef = ref(null)
const photoInputRef = ref(null)

const showPhotoCropper = ref(false)
const tempPhotoSrc = ref('')
const showImageDialog = ref(false)
const dialogImageSrc = ref('')

const handlePhotoUpload = (event) => {
  const file = event.target.files[0]
  if (file && file.type.startsWith('image/')) {
    const reader = new FileReader()
    reader.onload = (e) => {
      tempPhotoSrc.value = e.target.result
      showPhotoCropper.value = true
    }
    reader.readAsDataURL(file)
  }
}

const handleCropConfirm = (croppedImage) => {
  photoImage.value = croppedImage
  showPhotoCropper.value = false
  tempPhotoSrc.value = ''
  // 清空 input，允許重複選擇同一張圖片
  // if (photoInputRef.value) {
  //   photoInputRef.value.value = ''
  // }
}

const handleCropCancel = () => {
  showPhotoCropper.value = false
  tempPhotoSrc.value = ''
  // 清空 input，允許重複選擇同一張圖片
  if (photoInputRef.value) {
    photoInputRef.value.value = ''
  }
}

const downloadBadge = () => {
  if (badgeRef.value) {
    badgeRef.value.downloadBadge((imageDataUrl) => {
      // 收到圖片資料後顯示在 dialog 中
      dialogImageSrc.value = imageDataUrl
      showImageDialog.value = true
    })
  }
}

const closeImageDialog = () => {
  showImageDialog.value = false
  setTimeout(() => {
    dialogImageSrc.value = ''
  }, 300)
}
</script>

<template>
  <div class="md:px-12 px-5 py-10 bg-gray-100 min-h-screen" id="app">
    <div class="flex justify-center mb-10">
      <img :src="title" alt="OWHEECE Badge Generator" class="h-20" />
    </div>
    <div class="flex gap-10 flex-wrap justify-center">
      <!-- 左邊 表單 -->
      <div class="bg-white p-12 rounded-xl shadow-xl md:flex-none h-full">
        <div class="form-group">
          <label for="name">姓名：</label>
          <input id="name" v-model="employeeName" type="text" placeholder="請輸入姓名" />
        </div>

        <div class="form-group">
          <label for="id">部門：</label>
          <input id="id" v-model="employeeId" type="text" placeholder="請輸入部門" />
        </div>

        <div class="form-group">
          <label for="photo">照片：</label>
          <input id="photo" ref="photoInputRef" type="file" accept="image/*" @change="handlePhotoUpload"
            class="file:mr-4 file:rounded-full file:border-0 file:bg-violet-50 file:px-4 file:py-2 file:text-sm file:font-semibold file:text-violet-700 hover:file:bg-violet-100 dark:file:bg-violet-600 dark:file:text-violet-100 dark:hover:file:bg-violet-500" />
          <small class="hint">上傳照片</small>
        </div>

        <button @click="downloadBadge" class="download-btn">下載員工證</button>
      </div>
      <!-- 右邊 預覽窗 -->
      <EmployeeBadge class="flex-1 h-[70vh] md:h-[75svh] p-12 rounded-xl shadow-xl bg-white" ref="badgeRef"
        :employee-name="employeeName" :employee-id="employeeId" :background-image="backgroundImage"
        :photo-image="photoImage" />
    </div>

    <ImageCropper :visible="showPhotoCropper" :image-src="tempPhotoSrc" title="裁切個人照片" :aspect-ratio="5 / 6"
      @confirm="handleCropConfirm" @cancel="handleCropCancel" />

    <!-- 圖片顯示 Dialog -->
    <div v-if="showImageDialog" class="image-dialog" @click="closeImageDialog">
      <div class="dialog-content" @click.stop>
        <div class="dialog-header">
          <h3>長按圖片儲存</h3>
          <button @click="closeImageDialog" class="close-btn">✕</button>
        </div>
        <!-- 高度50vh，要顯示滾動條 -->
        <div class="p-12 max-h-[70vh] overflow-auto ">
          <img :src="dialogImageSrc" alt="員工證" class="dialog-image" />
        </div>
        <div class="dialog-footer">
          <p>長按圖片選擇「儲存到照片」或「加入照片」等</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
#app {
  min-height: 100vh;
  /* background-color: #C5CFEA; */
}

.badge-preview {
  max-height: 80svh;
  /* 維持長寬比 */
  aspect-ratio: 638 / 1012;
}

.content-wrapper {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

@media (min-width: 768px) {
  .content-wrapper {
    flex-direction: row;
    align-items: flex-start;
  }
}

.form-container {
  background: #f3f4f6;
  padding: 30px;
  border-radius: 12px;
  flex: 1;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  font-weight: bold;
  margin-bottom: 8px;
  color: #374151;
  font-size: 16px;
}

.form-group input {
  width: 100%;
  padding: 12px;
  font-size: 16px;
  border: 2px solid #d1d5db;
  border-radius: 8px;
  box-sizing: border-box;
  transition: border-color 0.3s;
}

.form-group input:focus {
  outline: none;
  border-color: #2563eb;
}

.file-input {
  padding: 8px !important;
  cursor: pointer;
}

.hint {
  display: block;
  margin-top: 5px;
  color: #6b7280;
  font-size: 14px;
}

.download-btn {
  width: 100%;
  padding: 14px;
  font-size: 18px;
  font-weight: bold;
  color: white;
  background: #2563eb;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s;
}

.download-btn:hover {
  background: #1d4ed8;
}

.download-btn:active {
  transform: scale(0.98);
}

/* Dialog 樣式 */
.image-dialog {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.dialog-content {
  background: white;
  border-radius: 12px;
  max-width: 90%;
  max-height: 90vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
  from {
    transform: translateY(50px);
    opacity: 0;
  }

  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #e5e7eb;
}

.dialog-header h3 {
  margin: 0;
  color: #1f2937;
  font-size: 20px;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  color: #6b7280;
  cursor: pointer;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  transition: all 0.2s;
}

.close-btn:hover {
  background: #f3f4f6;
  color: #1f2937;
}

.dialog-body {
  padding: 20px;
  overflow: auto;
  display: flex;
  justify-content: center;
  align-items: center;
}

.dialog-image {
  max-width: 100%;
  height: auto;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  /* 圖片自動縮小放入容器 */
  object-fit: contain;
}

.dialog-footer {
  padding: 15px 20px;
  background: #f9fafb;
  border-top: 1px solid #e5e7eb;
}

.dialog-footer p {
  margin: 0;
  text-align: center;
  color: #2563eb;
  font-weight: 500;
  font-size: 14px;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {

  0%,
  100% {
    opacity: 1;
  }

  50% {
    opacity: 0.7;
  }
}

@media (min-width: 768px) {
  .dialog-content {
    max-width: 500px;
  }
}
</style>
