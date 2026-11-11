# 发布记录

## v1.1.0 — 2026-11-11

功能版本：在**现有帖子卡下方**追加一张「英文 → 中文」的 X 风格翻译帖子卡。
硬约束：**现有 DOM、CSS 选择器、JS 分支、导出范围、教案 1～6 课一行都不改**，全部为纯追加。

### 需求与界面依据

联网查证 X / Twitter 官方帮助中心、中文教程、2026 自动翻译报道与开源复刻项目后，确认两代翻译形态：

| | 经典手动版 | 2026 自动翻译版 |
| --- | --- | --- |
| 译文默认 | 隐藏 | 显示 |
| 触发 | 正文下方蓝色「翻译帖子」链接 | 自动 |
| 标注 | 展开后「由 Google 翻译自英语」 | 顶部「翻译自英语 · 由 Google 翻译」 |
| 切回原文 | 收起译文 | 「显示原文」链接 |

两种形态都写进 DOM，用 `hidden` 切换、不重建节点；左侧「演示形态」按钮组做 GUI 切换，默认经典版。

### 新增内容

- 新卡 `#translationCard`：完整 X 帖子卡外观（头像 / 昵称 / handle / 认证 / 时间 / 正文 / 译文 / 简化互动行），放在 `#postCard` 之后。
- 左侧新增「翻译卡（新增）」编辑组：**英文原文 / 中文译文两栏都可实时编辑**，中文输入法合成期复用现有 `composing` 保护。
- 交互：经典版「翻译帖子 → 隐藏翻译」；自动版「显示原文 → 显示翻译」，切换形态时交互状态自动复位。
- 跟随：深色 / 浅色主题、结构标注（独立 `.tr-region` 配色）、昵称 / handle / 认证 / 头像、视图切换，全部自动同步。
- 教案追加第 7 课「不许改旧代码时怎么接新功能」+ 常见坑速查表 3 行；第 1～6 课未改。

### 关键决策：怎么做到「现有零改动」

| 问题 | 做法 | 为什么 |
| --- | --- | --- |
| 导出不能变 | 新卡做成 `#postCard` 的**兄弟节点** | `buildExportSvg()` 只克隆 `[profileCard, postCard]`，兄弟节点天然不进 PNG，无需改导出代码 |
| CSS 不能覆盖 | 新规则一律 `tr-` 前缀，颜色走现有主题变量 | 主题自动跟随，旧规则一条不改 |
| 要接旧的昵称 / 头像数据 | `addEventListener` 事件叠加 + `MutationObserver` 观察 `avatarImg` | 不改 `render()` / `setAvatarImage()`；三种头像来源全被覆盖 |
| 视图切换要带上新卡 | 给 `[data-view-btn]` 再叠一个监听器 | 原 `viewButtons` 逻辑一行不改 |
| 结构标注不打架 | 独立 `.tr-region` + `--tr-region-color` | 不挤进现有 6 个 `.region` 配色 |

### 验证（本地 Playwright + 系统 Edge）

- 原有 9 项 Handle 打码回归：**9/9 PASS**
- v1.1.0 专项 18 项：**18/18 PASS**（合计 27 项，页面无 JS 报错）
- **现有零改动证据**：与 `v1.0.1` 快照逐项对比，`#postCard` / `#profileCard` / 两条工具条 / 底部说明 / 固定声明 / 编辑区旧 3 组 outerHTML **完全一致**；CSS 旧 122 条规则原样保留，新 36 条只追加在末尾
- **导出零变化铁证**：v1.0.1 与 v1.1.0 同参数导出 PNG 均为 `1324×598`，像素差 **0**，sha1 同为 `453d8eda3e9154dfa38b400a53018aa6f8f8b37b`
- 竖屏 375 / 横屏 1440 双尺寸验证：新卡未引入任何新溢出

### ⚠️ 已知既有项（本次未处理）

- `v1.0.1` 在 375 竖屏本就有 **5px 横向溢出**（`.editor` / `.stage` / `.toolbar` / `.preview` 老容器），v1.1.0 溢出元素列表与旧版完全一致。按「现有不要动」约束本次**不修**，留待单独补丁拍板。
- 翻译卡**不进入导出 PNG**（采用最保守的 A 方案），仅用于课堂演示；若要导出，建议新增第三档范围而不是改 `clones`。

### 改动文件

| 文件 | 变化 |
| --- | --- |
| `index.html` | 77,601 → 97,591 bytes；新增 CSS 36 条 / 编辑组 1 个 / 翻译卡 1 张 / JS 段 1 个，全部纯追加 |
| `教案.md` | 追加第 7 课 + 速查表 3 行 |
| `README.md` | 功能 / 教学用法 / 技术实现 / 版本行同步到 v1.1.0 |
| `RELEASE_NOTES.md` | 本节 |

### 在线地址

- GitHub Pages：https://openclawzhangchong.github.io/x-post-teaching/
- here.now：https://frosty-lichen-w688.here.now/

### 回滚

`git checkout v1.1.0`（annotated tag）；回到纯旧版用 `git checkout v1.0.1`

## v1.0.1 — 2026-11-11

补丁版本：消除静态部署下的控制台 404。

- 现象：GitHub Pages 部署后浏览器自动请求 `https://openclawzhangchong.github.io/favicon.ico`，返回 404，控制台出现一条资源错误（与页面逻辑无关，功能全部正常）。
- 修法：`index.html` 的 `<head>` 增加一行内联 base64 SVG favicon，浏览器不再发起默认 favicon 请求；仍是单文件、无外链。
- 改动位置：`index.html` head 区 1 行；`README.md` 版本行；`RELEASE_NOTES.md` 本节。
- 复验：本地 / GitHub Pages 自测恢复 9/9 PASS，控制台无报错。

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
