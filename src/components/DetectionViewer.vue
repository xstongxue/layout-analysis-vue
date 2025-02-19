<template>
  <div class="detection-viewer">
    <div class="image-container" v-if="image">
      <img :src="imageUrl" class="preview-image" ref="previewImage" @load="onImageLoad" />
      <!-- Detection Boxes -->
      <div v-for="(box, index) in detectionBoxes" :key="index" class="detection-box" :style="getBoxStyle(box)"
        @mousedown="startDrag(index, $event)">
        <div class="box-label">
          <span class="box-type">{{ box.type }}</span>
          <span class="box-confidence">{{ (box.confidence * 100).toFixed(0) }}%</span>
        </div>
        <!-- Resize Handles -->
        <div class="resize-handle top-left" @mousedown.stop="startResize(index, 'top-left', $event)"></div>
        <div class="resize-handle top-right" @mousedown.stop="startResize(index, 'top-right', $event)"></div>
        <div class="resize-handle bottom-left" @mousedown.stop="startResize(index, 'bottom-left', $event)"></div>
        <div class="resize-handle bottom-right" @mousedown.stop="startResize(index, 'bottom-right', $event)"></div>
      </div>
    </div>
    <div v-else class="no-image">
      请先上传图片以进行检测。
    </div>
  </div>
</template>

<script>
export default {
  name: 'DetectionViewer',
  props: {
    image: {
      type: [File, null],
      default: null,
    },
    detectionBoxes: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      imageUrl: null,
      imageWidth: 0,
      imageHeight: 0,
      scaleX: 1,
      scaleY: 1,
      draggingBoxIndex: null,
      dragOffset: { x: 0, y: 0 },
      resizingBoxIndex: null,
      resizeDirection: null,
      initialBox: null,
      initialMousePosition: null,
    };
  },
  watch: {
    image: {
      immediate: true,
      handler(newImage) {
        if (newImage) {
          this.imageUrl = URL.createObjectURL(newImage);
        } else {
          this.imageUrl = null;
          this.imageWidth = 0;
          this.imageHeight = 0;
        }
      },
    },
  },
  mounted() {
    document.addEventListener('mousemove', this.handleMouseMove);
    document.addEventListener('mouseup', this.stopDragOrResize);
  },
  beforeUnmount() {
    document.removeEventListener('mousemove', this.handleMouseMove);
    document.removeEventListener('mouseup', this.stopDragOrResize);
  },
  methods: {
    onImageLoad() {
      this.imageWidth = this.$refs.previewImage.naturalWidth;
      this.imageHeight = this.$refs.previewImage.naturalHeight;
      this.calculateScale();
    },
    calculateScale() {
      const imageElement = this.$refs.previewImage;
      this.scaleX = imageElement.offsetWidth / this.imageWidth;
      this.scaleY = imageElement.offsetHeight / this.imageHeight;
    },
    getBoxStyle(box) {
      return {
        left: `${box.x * this.scaleX}px`,
        top: `${box.y * this.scaleY}px`,
        width: `${box.width * this.scaleX}px`,
        height: `${box.height * this.scaleY}px`,
        border: `2px solid ${this.getBoxColor(box.confidence)}`,
        backgroundColor: `${this.getBoxColor(box.confidence, 0.1)}`,
        position: 'absolute',
        cursor: this.draggingBoxIndex !== null || this.resizingBoxIndex !== null ? 'grabbing' : 'grab',
      };
    },
    getBoxColor(confidence, alpha = 1) {
      const hue = confidence * 120;
      return `hsla(${hue}, 100%, 50%, ${alpha})`;
    },
    startDrag(index, event) {
      this.draggingBoxIndex = index;
      this.resizingBoxIndex = null;
      this.initialBox = { ...this.detectionBoxes[index] };
      const imageElement = this.$refs.previewImage;
      this.dragOffset = {
        x: event.clientX - imageElement.offsetLeft - this.initialBox.x * this.scaleX,
        y: event.clientY - imageElement.offsetTop - this.initialBox.y * this.scaleY,
      };
    },
    handleMouseMove(event) {
      if (this.draggingBoxIndex !== null) {
        this.dragBox(event);
      } else if (this.resizingBoxIndex !== null) {
        this.resizeBox(event);
      }
    },
    dragBox(event) {
      if (this.draggingBoxIndex === null) return;

      const imageElement = this.$refs.previewImage;
      let newX = (event.clientX - imageElement.offsetLeft - this.dragOffset.x) / this.scaleX;
      let newY = (event.clientY - imageElement.offsetTop - this.dragOffset.y) / this.scaleY;

      newX = Math.max(0, Math.min(newX, this.imageWidth - this.initialBox.width));
      newY = Math.max(0, Math.min(newY, this.imageHeight - this.initialBox.height));

      this.detectionBoxes[this.draggingBoxIndex] = {
        ...this.initialBox,
        x: newX,
        y: newY,
      };
    },
    startResize(index, direction, event) {
      this.resizingBoxIndex = index;
      this.resizeDirection = direction;
      this.draggingBoxIndex = null;
      this.initialBox = { ...this.detectionBoxes[index] };
      const imageElement = this.$refs.previewImage;
      this.initialMousePosition = {
        x: event.clientX - imageElement.offsetLeft,
        y: event.clientY - imageElement.offsetTop,
      };
    },
    resizeBox(event) {
      if (this.resizingBoxIndex === null) return;

      const box = this.initialBox;
      let newWidth = box.width;
      let newHeight = box.height;
      let newX = box.x;
      let newY = box.y;

      const currentMouseX = event.clientX - this.$refs.previewImage.offsetLeft;
      const currentMouseY = event.clientY - this.$refs.previewImage.offsetTop;

      const deltaX = (currentMouseX - this.initialMousePosition.x) / this.scaleX;
      const deltaY = (currentMouseY - this.initialMousePosition.y) / this.scaleY;

      switch (this.resizeDirection) {
        case 'top-left':
          newWidth = box.width - deltaX;
          newHeight = box.height - deltaY;
          newX = box.x + deltaX;
          newY = box.y + deltaY;
          break;
        case 'top-right':
          newWidth = box.width + deltaX;
          newHeight = box.height - deltaY;
          newY = box.y + deltaY;
          break;
        case 'bottom-left':
          newWidth = box.width - deltaX;
          newHeight = box.height + deltaY;
          newX = box.x + deltaX;
          break;
        case 'bottom-right':
          newWidth = box.width + deltaX;
          newHeight = box.height + deltaY;
          break;
      }

      // Enforce minimum dimensions and image boundaries
      const minSize = 20; // Minimum width/height
      if (newWidth < minSize) {
        newX += newWidth - minSize;
        newWidth = minSize;
      }
      if (newHeight < minSize) {
        newY += newHeight - minSize;
        newHeight = minSize;
      }

      if (newX < 0) {
        newWidth += newX;
        newX = 0;
      }
      if (newY < 0) {
        newHeight += newY;
        newY = 0;
      }

      if (newX + newWidth > this.imageWidth) {
        newWidth = this.imageWidth - newX;
      }
      if (newY + newHeight > this.imageHeight) {
        newHeight = this.imageHeight - newY;
      }

      this.detectionBoxes[this.resizingBoxIndex] = {
        ...this.initialBox,
        x: newX,
        y: newY,
        width: newWidth,
        height: newHeight,
      };
    },
    stopDragOrResize() {
      this.draggingBoxIndex = null;
      this.resizingBoxIndex = null;
      this.initialBox = null;
      this.initialMousePosition = null;
    },
  },
};
</script>

<style scoped>
/* Detection Viewer container */
.detection-viewer {
  width: 100%;
  height: 100%;
  overflow: auto;
  background: #fff;
  border-radius: 4px;
  position: relative;
}

/* Image container */
.image-container {
  position: relative;
  display: inline-block;
}

/* Preview image styling */
.preview-image {
  display: block;
  max-width: 100%;
  max-height: 900px;
  object-fit: contain;
}

/* Detection box styling */
.detection-box {
  position: absolute;
  pointer-events: auto;
  box-sizing: border-box;
  cursor: grab;
}

/* Box label styling */
.box-label {
  position: absolute;
  top: -24px;
  left: 0;
  white-space: nowrap;
  font-size: 12px;
  pointer-events: none;
}

.box-type {
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 2px 6px;
  border-radius: 2px;
  margin-right: 4px;
}

.box-confidence {
  background-color: rgba(33, 150, 243, 0.7);
  color: white;
  padding: 2px 6px;
  border-radius: 2px;
}

/* No image styling */
.no-image {
  color: #999;
  text-align: center;
  padding: 20px;
}

/* Resize handle styling */
.resize-handle {
  position: absolute;
  width: 10px;
  height: 10px;
  background: rgba(76, 175, 80, 0.5);
  border: 1px solid #4CAF50;
  box-sizing: border-box;
  z-index: 10;
}

.resize-handle:hover {
  background: #4CAF50;
  cursor: nwse-resize;
}

.resize-handle.top-left {
  top: -5px;
  left: -5px;
  cursor: nwse-resize;
}

.resize-handle.top-right {
  top: -5px;
  right: -5px;
  cursor: nesw-resize;
}

.resize-handle.bottom-left {
  bottom: -5px;
  left: -5px;
  cursor: nesw-resize;
}

.resize-handle.bottom-right {
  bottom: -5px;
  right: -5px;
  cursor: nwse-resize;
}
</style>