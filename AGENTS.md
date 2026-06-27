# AGENTS.md

## 项目概览
- **项目名称**：变声还原器 (Voice Changer Restorer)
- **技术栈**：原生 HTML + CSS + JavaScript（单文件应用）
- **模板**：native-static
- **运行方式**：Python HTTP Server（`.coze` 配置）

## 项目结构
```
.
├── index.html          # 主页面（所有 HTML/CSS/JS 均在此单文件中）
├── .coze               # 构建与运行配置
├── DESIGN.md           # 设计规范
└── AGENTS.md           # 本文件
```

## 构建与运行
- **开发运行**：`python -m http.server ${DEPLOY_RUN_PORT} --bind 0.0.0.0`
- **端口**：从 `DEPLOY_RUN_PORT` 环境变量读取（默认 5000）
- **无构建步骤**：纯静态文件，无需编译

## 核心功能模块（index.html 内）

### 文件上传
- 拖拽上传 + 点击选择
- 支持格式：MP3 / WAV / M4A / OGG
- 使用 `File.arrayBuffer()` + `AudioContext.decodeAudioData()` 解码

### 预设变声类型
- `presets` 对象定义各预设参数（pitch / formant / speed）
- 预设：monkey、loli、uncle、robot、reverb、custom
- 点击预设按钮自动设置滑块值并触发处理

### 智能校准模式
- 上传一对音频：原声 + 变声后音频
- 自动分析音高偏移（自相关法基频检测）
- 自动分析共振峰偏移（频谱质心对比）
- 自动分析速度变化（时长对比）
- 显示置信度评分，一键应用校准参数

### 音频处理引擎
- 使用 `OfflineAudioContext` 离线渲染
- 音高变换：通过 `source.playbackRate` 实现（`2^(-semitones/12)`）
- 共振峰：通过 BiquadFilter (peaking) 链实现频谱重塑
- 机器人还原：Comb filter 抵消 vocoder 效果
- 混响还原：Highpass filter 减少低频混响

### 波形可视化
- 静态波形：Canvas 绘制音频波形（对称柱状图）
- 实时波形：AnalyserNode + requestAnimationFrame 驱动柱状频谱动画
- 支持窗口 resize 自适应

### 播放控制
- 原声播放 / 还原音频播放，可切换对比
- 播放进度条实时更新
- 霓虹发光反馈当前播放状态

### WAV 导出
- `bufferToWav()` 函数将 AudioBuffer 编码为 16-bit PCM WAV
- 使用 Blob URL + `<a>` download 触发下载

## 代码风格
- 使用 IIFE 封装避免全局变量污染
- `$()` 快捷选择器函数
- 所有 DOM 引用集中在文件顶部
- 事件监听使用箭头函数

## 注意事项
- AudioContext 在用户首次交互时创建（浏览器自动播放策略）
- 离线渲染是异步的（`startRendering()` 返回 Promise）
- Canvas 使用 2x 分辨率适配 Retina 屏幕
- 移动端使用 `clamp()` 和媒体查询适配
