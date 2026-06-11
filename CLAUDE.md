# earth-online · 地球Online 开发指南

## 项目概览
单文件 Web 应用 (`index.html`)，约 4000+ 行，包含 HTML/CSS/JS 三部分。
- **定位**：每日随机挑战 + 3D 地球探索 + 生命日历统计
- **设计风格**：深色背景 + 毛玻璃卡片（glassmorphism）
- **部署**：GitHub Pages（`main` 分支 → 自动部署）
- **无构建工具**：直接用原生 HTML/CSS/JS，零依赖框架

## 代码结构
```
index.html
  ├─ <style>   CSS（~1100 行）：变量定义、组件样式、动画
  ├─ <body>    HTML（~200 行）：视图结构
  └─ <script>  JS（~3000 行）：状态管理、交互逻辑、Three.js
```

### 关键区域
| 行号范围 | 内容 |
|---------|------|
| ~1-1100 | CSS 变量 + 全局样式 |
| ~2480-2505 | 探索视图 HTML（地球容器） |
| ~2840-2880 | 默认状态 DEFAULT_STATE |
| ~3050-3100 | 任务池 TASK_POOL + 换一换逻辑 |
| ~3480-3880 | **Three.js 3D 地球**（核心模块） |
| ~3820-3950 | 设置页 + Life Widget |

## 技术要点

### Three.js 地球（探索页）
- CDN：`three@0.160.0`
- 材质：`MeshPhongMaterial` + specular map（海洋反光）
- 纹理来源：`threejs.org/examples/textures/planets/`
- 性能基准（移动端）：
  - 像素比上限：1.5
  - 地球分段：64×48
  - 云层/大气分段：48×36
  - **新增 3D 物体时优先用低分段，保持总顶点数可控**

### CSS 设计系统
- 全部颜色通过 CSS 变量控制（`:root` 中定义）
- 毛玻璃效果：`var(--glass-bg)` + `backdrop-filter: blur()`
- 圆角体系：`--radius-xs` / `--radius-sm` / `--radius-md` / `--radius-lg` / `--radius-xl` / `--radius-pill`
- **新增 UI 元素一律使用已有变量，不硬编码颜色**

### 状态管理
- `appState` 对象存储在 `localStorage`，key 为 `earth-online-state`
- 每日任务基于日期 seed（`getDailySeed()`），保证同一天所有人看到相同任务
- 换一换逻辑：跟踪当天所有已见任务（`task-seen-{date}`），避免重复

## 开发注意事项

### 移动端兼容
- **所有 UI 改动必须在 iPhone 尺寸（375×812）下测试**
- 使用 `var(--safe-top)` / `var(--safe-bottom)` 适配刘海屏
- 底部导航栏高度约 70-80px，全局视口元素需预留空间
- 3D 地球的相机位置影响移动端显示，修改后需在真机验证

### Git 推送
- 网络环境可能需要 HTTP/1.1：`git -c http.version=HTTP/1.1 push origin main`
- Commit message 使用中文，格式：`模块名：简述改动`

### 提交前自检
1. `node --check` 验证 JS 语法（整个 `<script>` 块提取后检查）
2. 查看 diff：`git diff --stat`
3. 确认没有遗留 `console.log` 调试代码
4. 新增 CSS 是否使用了已有 CSS 变量

## 设计偏好（来自用户反馈）

### 前端风格
- 偏好：干净、高质感、写实风格（非卡通/廉价 3D 感）
- 参考：Apple Maps 卫星模式、高德地图 3D 地球
- 动画：简洁、流畅，避免复杂多阶段动画
- 日历：偏好简约现代设计（如点阵日历）

### 常见反馈
- "有点廉价 3D 感" → 避免 toon/cel-shading，用写实 PBR/Phong 材质
- "感觉卡卡的" → 降低 3D 分段数、像素比，简化材质
- "被导航栏遮住了" → 调整元素在视口中的位置
- "没有之前好看" → 尊重用户审美，改动前先沟通方向
