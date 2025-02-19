<template>
  <div class="ocr-results">
    <div v-if="ocrResults.length" class="results-container">
      <div v-for="(result, index) in ocrResults" :key="index" class="result-item">
        <div class="result-header">
          <span class="result-index">{{ index + 1 }}</span>
          <button class="copy-btn" @click="copyText(result)">复制</button>
        </div>
        <p class="result-text">{{ result }}</p>
      </div>
    </div>
    <div v-else class="no-results">
      <p>暂无 OCR 结果</p>
      <p class="hint">上传图片并进行检测后，点击 OCR 按钮开始识别</p>
    </div>
    <!-- Toast message -->
    <div v-if="toastMessage" class="toast">
      {{ toastMessage }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'OcrResults',
  props: {
    ocrResults: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      toastMessage: null,
    };
  },
  methods: {
    copyText(text) {
      navigator.clipboard.writeText(text)
        .then(() => {
          this.showToast('已复制到剪贴板');
        })
        .catch(err => {
          console.error('复制失败:', err);
          this.showToast('复制失败，请手动复制');
        });
    },
    showToast(message) {
      this.toastMessage = message;
      setTimeout(() => {
        this.toastMessage = null;
      }, 3000);
    },
  },
};
</script>

<style scoped>
/* OCR Results container */
.ocr-results {
  width: 100%;
  height: 100%;
  overflow-y: auto;
  position: relative;
  /* For toast positioning */
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  /* Align items at the top */
  align-items: stretch;
  /* Stretch items to fill width */
}

/* Results container */
.results-container {
  padding: 10px;
  /* Increased padding */
  width: 100%;
  /* Fill the container */
}

/* Result item */
.result-item {
  margin-bottom: 15px;
  /* Increased margin */
  padding: 15px;
  /* Increased padding */
  background-color: #f8f8f8;
  border-radius: 8px;
  border: 1px solid #eee;
}

/* Result header */
.result-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
  /* Increased margin */
}

/* Result index */
.result-index {
  background-color: #4CAF50;
  color: white;
  width: 24px;
  /* Increased width */
  height: 24px;
  /* Increased height */
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  /* Increased font size */
  font-weight: bold;
}

/* Copy button */
.copy-btn {
  padding: 6px 12px;
  /* Increased padding */
  background-color: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  /* Increased font size */
  transition: background-color 0.3s;
}

/* Result text */
.result-text {
  margin: 0;
  white-space: pre-wrap;
  word-break: break-all;
  font-size: 14px;
  /* Increased font size */
  line-height: 1.6;
  /* Increased line height */
  color: #333;
}

/* No results */
.no-results {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #666;
  font-size: 1em;
  /* Increased font size */
  padding: 20px;
  /* Increased padding */
}

/* Hint text */
.hint {
  font-size: 12px;
  /* Increased font size */
  color: #999;
  margin-top: 8px;
  /* Increased margin */
}

/* Toast message */
.toast {
  position: absolute;
  bottom: 20px;
  /* Increased bottom */
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 10px 20px;
  /* Increased padding */
  border-radius: 5px;
  z-index: 1001;
  font-size: 14px;
  /* Increased font size */
}
</style>
