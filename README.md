<file path="README.md">

## 简介

本项目是一个基于 Vue 3 和 Vite 的高效文档版面分析系统。它允许用户上传文档图片，自动检测版面元素，并使用 OCR 技术识别文本内容。

## 功能演示

![演示动画](/show.gif)

## 功能特性

- **图像上传:** 支持 JPG、PNG 和 BMP 格式的图像上传。
- **版面检测:** 自动检测图像中的版面元素，如文本块、图片等。
- **OCR 文本识别:** 使用 OCR 技术识别检测到的文本内容。
- **结果展示:** 清晰展示原始图像、检测结果和 OCR 文本。
- **交互式调整:** 允许用户拖动和调整检测框，以优化识别效果。
- **一键复制:** 方便地将识别的文本复制到剪贴板。

## 技术栈

- Vue 3
- Vite
- JavaScript
- HTML
- CSS

## 后端接口

本项目依赖以下后端接口：

### 1. 版面检测接口(CPP后端)
**后端项目:** https://github.com/xstongxue/TensorRT-Alpha-Plus 
- **URL:** `http://192.168.4.9:8888/pic_infer`
- **方法:** POST
- **请求参数 (FormData):**
    - `img`:  上传的图像文件。
    - `subject`:  主题 (例如: 'history')。
    - `is_show`:  是否显示 (true/false)。
    - `img_name`:  图像名称。
- **响应格式 (JSON):**
    ```json
    {
      "data": {
        "json": [
          {
            "box": [left, top, right, bottom],
            "type": "text",
            "confidence": 0.95
          },
          // ... 更多检测结果
        ]
      }
    }
    ```
    - `box`:  检测框的坐标 (左上角 x, 左上角 y, 右下角 x, 右下角 y)。
    - `type`:  检测到的元素类型。
    - `confidence`:  置信度。

### 2. OCR 接口(Python后端)
**后端项目:** https://github.com/xstongxue/OCR_API
- **URL:** `http://192.168.4.9:8899/OCR-QWen2.5-7B-VL`
- **方法:** POST
- **请求参数 (JSON):**
    ```json
    {
      "image": "base64 编码的图像数据",
      "box": {
        "x": 0,
        "y": 0,
        "width": 宽度,
        "height": 高度
      }
    }
    ```
    - `image`:  Base64 编码的图像数据。
    - `box`:  裁剪区域的坐标和尺寸。
- **响应格式 (JSON):**
    ```json
    {
      "ocr_text": "识别的文本内容"
    }
    ```
    - `ocr_text`:  识别的文本内容。

**注意:**  请确保后端接口可用，并根据实际情况修改 `App.vue` 中的 `serverUrl` 和 `ocrUrl`。

## 快速开始

1.  **安装依赖:**

    ```bash
    npm install
    ```

2.  **启动开发服务器:**

    ```bash
    npm run dev
    ```

3.  **构建生产版本:**

    ```bash
    npm run build
    ```

## 使用说明

1.  **上传图片:** 点击“上传图片”按钮，选择要分析的文档图片。
2.  **自动裁剪:** 点击“自动裁剪”按钮，系统将自动检测图像中的版面元素。
3.  **调整检测框:** 拖动和调整检测框，以优化识别区域。
4.  **OCR 识别:** 点击“OCR”按钮，系统将识别检测框中的文本内容。
5.  **查看结果:** 在右侧面板查看 OCR 识别结果。
6.  **复制文本:** 点击复制按钮，将识别的文本复制到剪贴板。
7.  **重置:** 点击“重置”按钮，清空所有结果，重新开始。

## 项目结构

```
layout-analysis/
├── index.html          # HTML 入口文件
├── package.json        # 项目依赖和脚本
├── README.md           # 项目说明文档
├── vite.config.js      # Vite 配置文件
├── src/
│   ├── App.vue         # 主应用组件
│   ├── main.js         # JavaScript 入口文件
│   ├── style.css       # 全局样式
│   ├── components/     # Vue 组件
│   │   ├── ImageUploader.vue   # 图像上传组件
│   │   ├── DetectionViewer.vue # 检测结果查看组件
│   │   ├── OcrResults.vue      # OCR 结果显示组件
```

## 组件说明

- **App.vue:** 主应用组件，负责整体布局和组件协调。
- **ImageUploader.vue:** 图像上传组件，允许用户上传图像并进行验证。
- **DetectionViewer.vue:** 检测结果查看组件，显示带有检测框的图像，并允许用户进行调整。
- **OcrResults.vue:** OCR 结果显示组件，展示识别的文本内容，并提供复制功能。

## 贡献

欢迎参与项目贡献！请提交 Pull Request。

## 许可证

MIT
</file>
