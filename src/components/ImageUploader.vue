<template>
  <div class="image-uploader">
    <div class="upload-area" @drop.prevent="handleDrop" @dragover.prevent="handleDragOver"
      @dragleave.prevent="handleDragLeave" @click="triggerFileInput" :class="{ 'drag-over': isDragOver }">
      <img v-if="image" :src="getImageUrl(image)" alt="Uploaded Image" class="preview-image" />
      <div v-else class="upload-placeholder">
        <i class="upload-icon">📁</i>
        <p>点击或拖拽图片到此处上传</p>
        <p class="upload-hint">支持 JPG、PNG、BMP 格式，最大 10MB</p>
      </div>
    </div>
    <div class="image-actions" v-if="image">
      <button class="remove-btn" @click="removeImage">移除图片</button>
    </div>
    <input type="file" ref="fileInput" @change="handleFileChange" accept="image/jpeg,image/png,image/bmp"
      style="display: none" />
  </div>
</template>

<script>
export default {
  name: 'ImageUploader',
  props: {
    image: {
      type: [File, null],
      default: null,
    },
  },
  data() {
    return {
      isDragOver: false,
    };
  },
  methods: {
    triggerFileInput() {
      this.$refs.fileInput.click();
    },
    validateFile(file) {
      const validTypes = ['image/jpeg', 'image/png', 'image/bmp'];
      if (!validTypes.includes(file.type)) {
        alert('请上传 JPG、PNG 或 BMP 格式的图片');
        return false;
      }

      const maxSize = 10 * 1024 * 1024; // 10MB
      if (file.size > maxSize) {
        alert('图片大小不能超过10MB');
        return false;
      }

      return true;
    },
    handleFileChange(event) {
      const file = event.target.files[0];
      if (file && this.validateFile(file)) {
        this.$emit('image-uploaded', file);
      }
      event.target.value = ''; // 清空 input
    },
    handleDrop(event) {
      this.isDragOver = false;
      const file = event.dataTransfer.files[0];
      if (file && this.validateFile(file)) {
        this.$emit('image-uploaded', file);
      }
    },
    handleDragOver() {
      this.isDragOver = true;
    },
    handleDragLeave() {
      this.isDragOver = false;
    },
    getImageUrl(file) {
      return URL.createObjectURL(file);
    },
    removeImage() {
      this.$emit('image-removed');
    },
  },
};
</script>

<style scoped>
/* Image Uploader container */
.image-uploader {
  width: 100%;
  height: 300px;
  display: flex;
  flex-direction: column;
}

/* Upload area styling */
.upload-area {
  flex-grow: 1;
  border: 2px dashed #ddd;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  overflow: hidden;
  position: relative;
}

.upload-area:hover,
.drag-over {
  border-color: #4CAF50;
  background-color: rgba(76, 175, 80, 0.05);
}

/* Preview image styling */
.preview-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

/* Upload placeholder styling */
.upload-placeholder {
  text-align: center;
  color: #666;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100%;
}

.upload-icon {
  font-size: 48px;
  margin-bottom: 10px;
  display: block;
}

.upload-hint {
  font-size: 12px;
  color: #999;
  margin-top: 5px;
}

/* Image actions styling */
.image-actions {
  display: flex;
  justify-content: center;
  padding-top: 10px;
}

.remove-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  background-color: #f44336;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.remove-btn:hover {
  background-color: #d32f2f;
}
</style>
