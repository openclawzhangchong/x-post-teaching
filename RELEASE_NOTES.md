# 发布记录

## v1.0.0 — 2026-11-11

首个**公开版本**。此前均为本地迭代，本次归档开源并同时部署 GitHub Pages 与 here.now。

### 收录内容

- 单文件 `index.html`：X 帖子页前端教学复刻，原生 HTML / CSS / JS，双击即用
- `教案.md`：6 课教案 + 常见坑速查表
- `README.md`：功能说明、两个在线地址、教学用法、目录与版本信息
- `.nojekyll`：保证 GitHub Pages 原样发布静态文件

### 功能清单

实时编辑 / 深浅主题 / 响应式 / 结构标注 / 全部随机（11 字段）/ 恢复示例 / 2× PNG 导出（两种范围、三档边距）/ 头像三选一 / 配图上传 / **Handle 导出打码（默认关闭）**。

### 本次重点：Handle 打码

- 导出工具条新增 GUI 复选框「Handle 打码」，无 `checked` 属性即默认关闭
- 预览与导出同步；两处 `@Handle`（个人主页、帖子卡）一起处理
- 状态挂在 `.handle` 元素自身的 `is-masked` class 上 —— 导出用 `cloneNode(true)` + `foreignObject`，克隆 DOM 里没有 `body`，依赖祖先状态的选择器会导致 PNG 漏码
- 遮盖用透明文字 + `repeating-linear-gradient` 条纹，不用 `filter: blur()`；不改布局属性，打码前后零位移

### 验证

- Playwright（系统 Edge）自动化：本地与线上各 **9/9 PASS**，页面无 JS 报错
- 导出组合：深 / 浅主题 × 帖子卡 / 主页+帖子，4 种全部成功
- PNG 像素对比：差异像素 6381（0.8%），差异带高 39px、条纹跳变 12 次、导出尺寸不变
- 线上地址均返回 HTTP 200

### 在线地址

- GitHub Pages：https://openclawzhangchong.github.io/x-post-teaching/
- here.now：https://frosty-lichen-w688.here.now/

### 回滚

`git checkout v1.0.0`（annotated tag）
