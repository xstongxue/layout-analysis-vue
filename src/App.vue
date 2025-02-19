<template>
  <div class="app-container">

    <!-- Sidebar with Action Buttons -->
    <aside class="sidebar">
      <!-- 字体大小 -->
      <h2 style="font-size: 1em;">面向数字化题库建设的高效文档版面分析系统</h2>
      <h2 style="font-size: 1em;">创建者：GMS</h2>
      <div class="button-group">
        <button class="btn upload-btn" @click="triggerImageUpload">
          上传图片
        </button>
        <button class="btn detect-btn" @click="handleDetection" :disabled="!originalImage || isLoading">
          自动裁剪(版面分析)
        </button>
        <button class="btn ocr-btn" @click="handleOcr" :disabled="!detectionBoxes.length || isOcrLoading">
          OCR
        </button>
        <button v-if="originalImage" class="btn reset-btn" @click="resetAll">
          重置
        </button>
      </div>
    </aside>

    <!-- Main Content -->
    <main class="main-content">
      <!-- Left Panel: Image Uploader -->
      <div class="panel">
        <h2>原始图片</h2>
        <ImageUploader :image="originalImage" @image-uploaded="handleImageUpload" @image-removed="handleImageRemove" />
      </div>

      <!-- Middle Panel: Detection Viewer -->
      <div class="panel">
        <h2>检测结果</h2>
        <DetectionViewer :image="originalImage" :detectionBoxes="detectionBoxes" />
      </div>

      <!-- Right Panel: OCR Results -->
      <div class="panel">
        <h2>OCR结果</h2>
        <OcrResults :ocrResults="ocrResults" />
      </div>
    </main>

    <!-- Loading Overlay -->
    <div v-if="isLoading || isOcrLoading" class="loading-overlay">
      <div class="loading-spinner"></div>
      <div class="loading-text">
        {{ isLoading ? '正在分析图片...' : '正在进行文字识别...' }}
      </div>

      <!-- Status Message -->
      <div v-if="requestStatus" :class="['request-status', requestStatus.type]">
        {{ requestStatus.message }}
      </div>
    </div>
  </div>
</template>

<script>
import ImageUploader from './components/ImageUploader.vue';
import DetectionViewer from './components/DetectionViewer.vue';
import OcrResults from './components/OcrResults.vue';

export default {
  name: 'App',
  components: {
    ImageUploader,
    DetectionViewer,
    OcrResults,
  },
  data() {
    return {
      originalImage: null,
      detectionBoxes: [],
      ocrResults: [],
      isLoading: false,
      isOcrLoading: false,
      requestStatus: null,
      serverUrl: 'http://192.168.4.9:8888/pic_infer',
      ocrUrl: 'http://192.168.4.9:8899/OCR-QWen2.5-7B-VL',
      subject: 'history',
      isShow: false,
      panelHeight: 200, // Base panel height
      panelSpacing: 5, // Spacing between panels
    };
  },
  computed: {
    sidebarMaxHeight() {
      return (this.panelHeight * 3) + (this.panelSpacing * 2) + 'px';
    },
  },
  methods: {
    triggerImageUpload() {
      this.$refs.imageUploader.$refs.fileInput.click();
    },
    resetAll() {
      this.originalImage = null;
      this.detectionBoxes = [];
      this.ocrResults = [];
      this.requestStatus = null;
    },
    async handleImageUpload(file) {
      try {
        if (!file) {
          throw new Error('未选择图片');
        }

        const validTypes = ['image/jpeg', 'image/png', 'image/bmp'];
        if (!validTypes.includes(file.type)) {
          throw new Error('请上传 JPG、PNG 或 BMP 格式的图片');
        }

        if (file.size > 10 * 1024 * 1024) {
          throw new Error('图片大小不能超过 10MB');
        }

        this.resetAll();
        this.originalImage = file;
        this.showStatus('图片已就绪', 'success');
      } catch (error) {
        console.error('Upload error:', error);
        this.showStatus(error.message, 'error');
      }
    },
    handleImageRemove() {
      this.resetAll();
      this.showStatus('图片已移除', 'info');
    },
    async handleDetection() {
      if (!this.originalImage) {
        this.showStatus('请先上传图片', 'error');
        return;
      }

      this.isLoading = true;
      this.showStatus('正在进行目标检测...', 'info');

      try {
        const formData = new FormData();
        formData.append('img', this.originalImage, this.originalImage.name);
        formData.append('subject', this.subject);
        formData.append('is_show', this.isShow ? 'true' : 'false');
        formData.append('img_name', this.originalImage.name);

        const response = await fetch(this.serverUrl, {
          method: 'POST',
          body: formData,
        });

        if (!response.ok) {
          throw new Error(`目标检测服务响应错误 (${response.status})`);
        }

        const result = await response.json();
        if (!result || !result.data || !result.data.json) {
          throw new Error('无效的目标检测服务响应格式');
        }

        this.detectionBoxes = result.data.json.map(item => {
          const [left, top, right, bottom] = item.box;
          return {
            x: left,
            y: top,
            width: right - left,
            height: bottom - top,
            type: item.type,
            confidence: item.confidence,
          };
        }).filter(box => box.width > 0 && box.height > 0);


      } catch (error) {
        console.error('Detection error:', error);
        this.showStatus(`目标检测失败: ${error.message}`, 'error');
      } finally {
        this.isLoading = false;
        if (this.detectionBoxes.length > 0) {
          this.showStatus(`检测完成，找到 ${this.detectionBoxes.length} 个区域`, 'success');
        } else {
          this.showStatus('未检测到任何区域', 'info');
        }
      }
    },
    async handleOcr() {
      if (!this.originalImage) {
        this.showStatus('请先上传图片', 'error');
        return;
      }

      if (this.isOcrLoading) return;

      this.isOcrLoading = true;
      this.showStatus('正在进行文字识别...', 'info');

      try {
        // 1. 将图片转换为 Canvas，以便裁剪各个区域
        const imageUrl = URL.createObjectURL(this.originalImage);
        const img = await new Promise((resolve, reject) => {
          const image = new Image();
          image.onload = () => resolve(image);
          image.onerror = reject;
          image.src = imageUrl;
        });

        // 2. 处理每个检测框区域
        const ocrPromises = this.detectionBoxes.map(async (box) => {
          // 创建 Canvas 并设置大小
          const canvas = document.createElement('canvas');
          canvas.width = box.width;
          canvas.height = box.height;
          const ctx = canvas.getContext('2d');

          // 裁剪指定区域
          ctx.drawImage(
            img,
            box.x, box.y,
            box.width, box.height,
            0, 0,
            box.width, box.height
          );

          // 获取裁剪区域的 base64 数据
          const base64Image = canvas.toDataURL('image/jpeg').split(',')[1];

          // 发送 OCR 请求
          const response = await fetch(this.ocrUrl, {
            method: 'POST',
            headers: {
              "Content-Type": "application/json",
              'Accept': 'application/json',
            },
            body: JSON.stringify({
              image: base64Image,
              box: {
                x: 0,
                y: 0,
                width: box.width,
                height: box.height,
              },
            }),
          });

          if (!response.ok) {
            console.error(`OCR 服务响应错误 (${response.status}): ${await response.text()}`);
            return '识别失败';
          }

          const result = await response.json();
          return result?.ocr_text || '识别失败';
        });

        // 3. 保存所有区域的识别文本结果
        this.ocrResults = await Promise.all(ocrPromises);
        this.showStatus('文字识别完成', 'success');

      } catch (error) {
        console.error('OCR error:', error);
        this.showStatus(`文字识别失败: ${error.message}`, 'error');
      } finally {
        this.isOcrLoading = false;
      }
    },
    showStatus(message, type = 'info') {
      this.requestStatus = { message, type };
      setTimeout(() => {
        this.requestStatus = null;
      }, 3000);
    },
  },
};
</script>


<style scoped>
/* Overall layout */
.app-container {
  display: flex;
  /* min-height: 100vh; */
  background-color: #f5f5f5;
  padding: 5px;
  /* Reduced padding */
}


/* Sidebar styling */
.sidebar {
  width: 160px;
  /* Adjusted sidebar width */
  background: white;
  border-radius: 8px;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
  padding: 5px;
  /* Reduced padding */
  margin-right: 5px;
  /* Reduced spacing */
  display: flex;
  flex-direction: column;
  align-items: stretch;
}

/* Main content styling */
.main-content {
  display: flex;
  flex-direction: column;
  gap: 5px;
  /* Reduced spacing */
  flex: 1;
}

/* Panel styling */
.panel {
  background: white;
  border-radius: 8px;
  padding: 39px;
  /* Reduced padding */
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
  min-height: 50px;
  /* Reduced panel height */
}

/* Responsive panel layout */
@media (max-width: 767px) {
  .app-container {
    flex-direction: column;
  }

  .sidebar {
    width: 100%;
    margin: 0 0 5px 0;
  }

  .main-content {
    padding: 0;
  }
}

h2 {
  margin-top: 0;
  color: #333;
  border-bottom: 1px solid #eee;
  padding-bottom: 3px;
  font-size: 0.9em;
  /* Reduced font size */
}

/* Button group styling */
.button-group {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  gap: 3px;
  /* Reduced spacing */
}

/* Button styling */
.btn {
  padding: 6px 8px;
  border: none;
  border-radius: 6px;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
  font-size: 0.8em;
  white-space: nowrap;
  line-height: 1;
  text-align: center;
}

.upload-btn {
  background-color: #4CAF50;
}

.detect-btn {
  background-color: #2196F3;
}

.ocr-btn {
  background-color: #9C27B0;
}

.reset-btn {
  background-color: #607D8B;
}

.btn:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-1px);
}

.btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
  opacity: 0.7;
}

/* Loading overlay */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.loading-spinner {
  width: 30px;
  height: 30px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.loading-text {
  color: white;
  margin-top: 5px;
  font-size: 14px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

/* Status message styling */
.request-status {
  position: fixed;
  bottom: 10px;
  right: 10px;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 12px;
  z-index: 9999;
  opacity: 0.9;
  transition: all 0.3s ease;
}

.info {
  background-color: #2196F3;
  color: white;
}

.success {
  background-color: #4CAF50;
  color: white;
}

.error {
  background-color: #F44336;
  color: white;
}
</style>
