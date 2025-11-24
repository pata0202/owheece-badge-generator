<script setup>
import { ref } from 'vue'
import EmployeeBadge from './components/EmployeeBadge.vue'
import ImageCropper from './components/ImageCropper.vue'

const employeeName = ref('')
const employeeId = ref('')
const backgroundImage = ref('')
const photoImage = ref('')
const badgeRef = ref(null)

const showPhotoCropper = ref(false)
const tempPhotoSrc = ref('')

const handleImageUpload = (event) => {
  const file = event.target.files[0]
  if (file && file.type.startsWith('image/')) {
    const reader = new FileReader()
    reader.onload = (e) => {
      backgroundImage.value = e.target.result
    }
    reader.readAsDataURL(file)
  }
}

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
  // 清空 input，允許重複選擇同一張圖片
  event.target.value = ''
}

const handleCropConfirm = (croppedImage) => {
  photoImage.value = croppedImage
  showPhotoCropper.value = false
  tempPhotoSrc.value = ''
}

const handleCropCancel = () => {
  showPhotoCropper.value = false
  tempPhotoSrc.value = ''
}

const downloadBadge = () => {
  if (badgeRef.value) {
    badgeRef.value.downloadBadge()
  }
}
</script>

<template>
  <div class="badge-generator">
    <h1>員工證產生器</h1>
    
    <div class="content-wrapper">
      <div class="form-container">
        <div class="form-group">
          <label for="name">姓名：</label>
          <input 
            id="name"
            v-model="employeeName" 
            type="text" 
            placeholder="請輸入姓名"
          />
        </div>
        
        <div class="form-group">
          <label for="id">員工編號：</label>
          <input 
            id="id"
            v-model="employeeId" 
            type="text" 
            placeholder="請輸入員工編號"
          />
        </div>

        <div class="form-group">
          <label for="photo">個人照片：</label>
          <input 
            id="photo"
            type="file" 
            accept="image/*"
            @change="handlePhotoUpload"
            class="file-input"
          />
          <small class="hint">上傳個人照片</small>
        </div>

        <!-- <div class="form-group">
          <label for="background">背景圖片：</label>
          <input 
            id="background"
            type="file" 
            accept="image/*"
            @change="handleImageUpload"
            class="file-input"
          />
          <small class="hint">選擇性上傳背景圖片</small>
        </div> -->

        <button @click="downloadBadge" class="download-btn">下載員工證</button>
      </div>

      <EmployeeBadge 
        ref="badgeRef"
        :employee-name="employeeName" 
        :employee-id="employeeId"
        :background-image="backgroundImage"
        :photo-image="photoImage"
      />
    </div>

    <ImageCropper
      :visible="showPhotoCropper"
      :image-src="tempPhotoSrc"
      title="裁切個人照片"
      :aspect-ratio="5/6"
      @confirm="handleCropConfirm"
      @cancel="handleCropCancel"
    />
  </div>
</template>

<style>
#app {
  min-height: 100vh;
}

.badge-generator {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}

h1 {
  text-align: center;
  color: #1f2937;
  margin-bottom: 30px;
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
</style>
