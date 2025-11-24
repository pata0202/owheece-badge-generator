<template>
  <div v-if="visible" class="cropper-modal" @click.self="$emit('cancel')">
    <div class="cropper-container">
      <div class="cropper-header">
        <h3>{{ title }}</h3>
        <button @click="$emit('cancel')" class="close-btn">✕</button>
      </div>
      
      <div class="cropper-content">
        <Cropper
          ref="cropper"
          :src="imageSrc"
          :stencil-props="stencilProps"
          class="cropper"
        />
      </div>
      
      <div class="cropper-footer">
        <button @click="$emit('cancel')" class="btn btn-cancel">取消</button>
        <button @click="cropImage" class="btn btn-confirm">確認裁切</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { Cropper } from 'vue-advanced-cropper'
import 'vue-advanced-cropper/dist/style.css'

const props = defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  imageSrc: {
    type: String,
    required: true
  },
  title: {
    type: String,
    default: '裁切圖片'
  },
  aspectRatio: {
    type: Number,
    default: undefined
  }
})

const emit = defineEmits(['cancel', 'confirm'])

const cropper = ref(null)

const stencilProps = props.aspectRatio 
  ? { aspectRatio: props.aspectRatio }
  : {}

const cropImage = () => {
  if (cropper.value) {
    const { canvas } = cropper.value.getResult()
    if (canvas) {
      const croppedImage = canvas.toDataURL('image/png')
      emit('confirm', croppedImage)
    }
  }
}
</script>

<style scoped>
.cropper-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 20px;
}

.cropper-container {
  background: white;
  border-radius: 12px;
  width: 100%;
  max-width: 800px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.cropper-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  border-bottom: 1px solid #e5e7eb;
}

.cropper-header h3 {
  margin: 0;
  font-size: 20px;
  color: #1f2937;
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
  border-radius: 4px;
  transition: background 0.2s;
}

.close-btn:hover {
  background: #f3f4f6;
}

.cropper-content {
  flex: 1;
  padding: 20px;
  overflow: hidden;
  min-height: 400px;
}

.cropper {
  height: 100%;
  max-height: 500px;
}

.cropper-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 20px;
  border-top: 1px solid #e5e7eb;
}

.btn {
  padding: 10px 24px;
  font-size: 16px;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-cancel {
  background: #f3f4f6;
  color: #374151;
}

.btn-cancel:hover {
  background: #e5e7eb;
}

.btn-confirm {
  background: #2563eb;
  color: white;
}

.btn-confirm:hover {
  background: #1d4ed8;
}

.btn:active {
  transform: scale(0.98);
}

@media (max-width: 768px) {
  .cropper-modal {
    padding: 10px;
  }
  
  .cropper-content {
    min-height: 300px;
  }
  
  .cropper {
    max-height: 400px;
  }
  
  .cropper-footer {
    flex-direction: column;
  }
  
  .btn {
    width: 100%;
  }
}
</style>
