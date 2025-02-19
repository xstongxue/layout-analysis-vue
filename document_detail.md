<file path="detailed_documentation_zh.md">
# 组件详细文档

## 1. App.vue

**描述:**
主应用程序组件，用于协调整个文档版面分析系统。它包括图像上传器、检测查看器、OCR 结果显示和整体布局。

**功能:**

- **管理状态:** 管理原始图像、检测框和 OCR 结果的状态。
  ```vue
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
    };
  },
  ```

- **触发操作:** 提供用于触发图像上传、检测、OCR 和重置操作的按钮。
  ```vue
  <button class="btn upload-btn" @click="triggerImageUpload">上传图片</button>
  <button class="btn detect-btn" @click="handleDetection" :disabled="!originalImage || isLoading">自动裁剪</button>
  <button class="btn ocr-btn" @click="handleOcr" :disabled="!detectionBoxes.length || isOcrLoading">OCR</button>
  <button v-if="originalImage" class="btn reset-btn" @click="resetAll">重置</button>
  ```

- **显示加载:** 显示加载覆盖和状态消息。
  ```vue
  <div v-if="isLoading || isOcrLoading" class="loading-overlay">
    <div class="loading-spinner"></div>
    <div class="loading-text">{{ isLoading ? '正在分析图片...' : '正在进行文字识别...' }}</div>
  </div>
  ```

- **组件通信:** 处理子组件之间的通信。例如，处理 `ImageUploader` 发出的 `image-uploaded` 事件。
  ```vue
  <ImageUploader :image="originalImage" @image-uploaded="handleImageUpload" @image-removed="handleImageRemove" />
  ```

**使用的组件:**
- ImageUploader: 用于上传和显示原始图像。
- DetectionViewer: 用于显示带有检测框的图像。
- OcrResults: 用于显示 OCR 结果。

## 2. ImageUploader.vue

**描述:**
一个允许用户通过拖放或文件选择上传图像的组件。

**功能:**

- **图像上传:** 通过拖放或文件输入处理图像上传。
  ```vue
  <div class="upload-area" @drop.prevent="handleDrop" @dragover.prevent="handleDragOver" @click="triggerFileInput">
    ...
    <input type="file" ref="fileInput" @change="handleFileChange" style="display: none" />
  </div>
  ```

- **文件验证:** 验证上传的文件类型（JPG、PNG、BMP）和大小（最大 10MB）。
  ```javascript
  validateFile(file) {
    const validTypes = ['image/jpeg', 'image/png', 'image/bmp'];
    if (!validTypes.includes(file.type)) {
      alert('请上传 JPG、PNG 或 BMP 格式的图片');
      return false;
    }
    // ... (size validation)
  }
  ```

- **图像预览:** 显示上传图像的预览。
  ```vue
  <img v-if="image" :src="getImageUrl(image)" alt="Uploaded Image" class="preview-image" />
  ```

- **事件发出:** 在上传或删除图像时发出事件。
  ```javascript
  handleFileChange(event) {
    const file = event.target.files[0];
    if (file && this.validateFile(file)) {
      this.$emit('image-uploaded', file);
    }
  }
  ```

**属性:**
- image: 当前上传的图像文件（File 或 null）。

**发出事件:**
- image-uploaded: 当上传新图像时。
- image-removed: 当删除当前图像时。

## 3. DetectionViewer.vue

**描述:**
一个显示带有覆盖检测框的上传图像的组件。

**功能:**

- **图像显示:** 显示上传的图像。
  ```vue
  <img :src="imageUrl" class="preview-image" ref="previewImage" @load="onImageLoad" />
  ```

- **检测框渲染:** 根据提供的数据渲染检测框。
  ```vue
  <div v-for="(box, index) in detectionBoxes" :key="index" class="detection-box" :style="getBoxStyle(box)">
    ...
  </div>
  ```

- **拖拽调整:** 允许用户拖动检测框。
  ```javascript
  startDrag(index, event) {
    this.draggingBoxIndex = index;
    // ...
  }

  dragBox(event) {
    // ... (logic to update box position)
  }
  ```

- **缩放调整:** 允许用户调整检测框的大小。
  ```javascript
  startResize(index, direction, event) {
    this.resizingBoxIndex = index;
    this.resizeDirection = direction;
    // ...
  }

  resizeBox(event) {
    // ... (logic to update box dimensions)
  }
  ```

- **比例计算:** 计算比例因子以正确地在图像上定位框。
  ```javascript
  calculateScale() {
    const imageElement = this.$refs.previewImage;
    this.scaleX = imageElement.offsetWidth / this.imageWidth;
    this.scaleY = imageElement.offsetHeight / this.imageHeight;
  }
  ```

**属性:**
- image: 上传的图像文件（File 或 null）。
- detectionBoxes: 一个对象数组，每个对象代表一个检测框，具有 x、y、width、height、type 和 confidence 等属性。

## 4. OcrResults.vue

**描述:**
一个以列表格式显示 OCR 结果的组件。

**功能:**

- **结果显示:** 以格式化的列表显示 OCR 结果。
  ```vue
  <div v-for="(result, index) in ocrResults" :key="index" class="result-item">
    <p class="result-text">{{ result }}</p>
  </div>
  ```

- **复制功能:** 提供一个复制按钮，用于将文本复制到剪贴板。
  ```vue
  <button class="copy-btn" @click="copyText(result)">复制</button>
  ```
  ```javascript
  copyText(text) {
    navigator.clipboard.writeText(text)
      .then(() => {
        this.showToast('已复制到剪贴板');
      })
      .catch(err => {
        console.error('复制失败:', err);
        this.showToast('复制失败，请手动复制');
      });
  }
  ```

- **Toast 提示:** 复制文本时显示 Toast 消息。
  ```vue
  <div v-if="toastMessage" class="toast">{{ toastMessage }}</div>
  ```

- **无结果提示:** 当没有 OCR 结果时显示消息。
  ```vue
  <div v-else class="no-results">
    <p>暂无 OCR 结果</p>
  </div>
  ```

**属性:**
- ocrResults: 一个字符串数组，每个字符串代表一个 OCR 结果。
</file>
