# 发布记录

## v1.2.0 — 2026-11-11

功能版本：编辑面板分组导航 / 折叠 + 默认关闭的受限就地编辑；教案新增第 8 课。

### 需求与决策

- 用户反馈「左边这种下拉很麻烦」，选定 **V1（P0 吸顶分组导航 + P1 分组折叠）**，并追加「双击右侧帖子内容直接改」的大胆想法。
- 评审结论：**不做「干掉左侧、全面就地编辑」**（会破坏本页「一份数据 → 三处视图」的核心教学点，还要把 `render()` 改成双向）。改为 **V1 + 受限就地编辑实验**：GUI 开关默认关、只开放正文与译文、双击=编辑不做删除 / 清空、提交后走原 `input` 通道。

### 实现（仍严格「现有零改动」）

| 项 | 做法 |
| --- | --- |
| P0 导航 | 编辑面板顶部加 `#nvBar`（`<div>`，4 个胶囊 + 「全部收起」），`position:sticky` 吸顶；点击用 `editor.scrollTo` 直达并自动展开目标组，窄屏退回 `scrollIntoView` |
| P1 折叠 | 运行时给 `.ed-group` 加 `nv-collapsed` class，CSS 只隐藏标题以外的直接子元素；默认收起「共用信息」「主页头部」，展开「帖子」「翻译卡」 |
| 就地编辑 | `#ieToggle` 无 `checked` 属性（默认关）；仅 `#postBody` / `#trTranslatedClassic` / `#trAutoBody` 三处挂 `dblclick`（事件叠加，旧节点一个属性不加） |
| 编辑态 | `body` 下的独立 `.ie-overlay` 浮层 + `textarea`，**不给卡片挂 `contenteditable`**；复制目标 computed 排版避免双击跳字 |
| 反向同步 | 提交时 `input.value = 新值` + `dispatchEvent(new Event('input'))`，走老的 `render()` / `trRender()`，三处视图一起变 |
| 导出隔离 | 浮层不进 `buildExportSvg()` 的 clones；提示虚线框规则带 `body.ie-on` 前缀（克隆 DOM 无 body）；`document` 捕获阶段兜底提交，保证导出前拿到新文本 |
| 保护基线 | 新开关**不放进任何 `.toolbar`**、导航条用 `<div>` 不用 `<section>`，确保两条工具条与 `.editor > section` 的 outerHTML 仍是零改动对比项 |

- 改动位置：`index.html` 的 13/14 CSS 段（追加在 `</style>` 前）、`.editor` 内的 `#nvBar`、11/12 JS 段（追加在 10.9 初始化之后）—— **v1.0.1 的 DOM / CSS / JS / 导出一行未动**。
- 4 个既有回归脚本各补 1 行「展开全部分组」准备步骤（因字段默认收起），**断言逻辑一行未改**；旧版 v1.0.1 快照无 nv 逻辑，该行是空操作。

### 验证

- 新增 `verify_v120.js` 专项 **17/17 PASS**：导航结构 / 默认折叠 / 隐藏生效 / 点导航展开+高亮+滚动 / sticky 吸顶 / 标题切换 / 一键全收全展 / 开关默认关 / 浮层不碰卡片 DOM / 写回同步 / Esc 取消 / 译文双形态同步 / 显示原文时拒编 / 编辑态直接导出 / 关闭后导出干净 / 工具条零污染 / 无报错
- 本地回归：`verify_mask.js` **9/9**、`verify_counts.js` **9/9**、`verify_v110.js` **18/18**，合计 **53/53**
- **导出零变化铁证**：v1.0.1 与 v1.2.0 同参数 PNG 均 `1324×598`，像素差 **0**，sha1 同为 `453d8eda3e9154dfa38b400a53018aa6f8f8b37b`；旧 122 条 CSS 规则原样保留，新 57 条只追加末尾
- 自测中发现并修复的真实 bug：点导航后的平滑滚动触发 `scroll` 监听抢走高亮 → 加 `nvLockUntil` 锁定期
- 第二个真实 bug：双击打开浮层后立刻自己关闭 → `window` 捕获阶段的 `scroll` 监听把 `focus()` 引起的自动滚动当成「用户滚走了」。修法：`focus({preventScroll:true})` + 打开后 400ms 守卫窗口 + 过滤 textarea 自身滚动
- 第三个视觉问题：编辑提示条是浮层外侧窄条，与互动行数字叠成噪音 → 改为浮层内部整宽不透明底条，浮层本身加 `var(--card-bg)` 背景隔离底层内容
- 线上：GitHub Pages / here.now 各跑专项 + 三档回归

### 已知既有项（未处理）

同 v1.1.x：`v1.0.1` 在 375 竖屏本就有 5px 横向溢出（老容器），按「现有不要动」约束未修；翻译卡仍不进入导出 PNG；就地编辑默认关闭。

### 改动文件

| 文件 | 变化 |
| --- | --- |
| `index.html` | 100,162 → 114,331 bytes；13/14 CSS 段、`#nvBar`、11/12 JS 段，全为纯追加 |
| `README.md` | 功能 / 教学用法（新增第 8 课）/ 技术实现 / 版本行同步到 v1.2.0 |
| `教案.md` | 新增第 8 课「数据驱动 vs 就地编辑」+ 速查表补 6 行 |
| `RELEASE_NOTES.md` | 本节 |

### 在线地址

- https://openclawzhangchong.github.io/x-post-teaching/
- https://frosty-lichen-w688.here.now/

## v1.1.1 — 2026-11-11

补丁版本：补齐翻译卡互动行的数字，并与现有主数据同步。

### 现象与根因

- 用户反馈（截图）：翻译卡底部互动行只有 4 个纯图标，没有回复 / 转推 / 点赞 / 查看 / 收藏的数字，看起来不像一条完整帖子。
- 根因：v1.1.0 把翻译卡互动行做成了「简化版」占位（4 个图标 + 独立 `.tr-action` 样式），既没补齐 6 个操作，也没接现有 5 个数字输入框（`replyInput` / `repostInput` / `likeInput` / `viewInput` / `bookmarkInput`）—— 属于**功能遗漏**，不是渲染故障。

### 修法（仍严格「现有零改动」）

| 项 | 做法 |
| --- | --- |
| 结构 | 互动行补齐为与现有帖子卡一致的 **6 个操作 + 5 个数字**（回复 / 转推 / 点赞 / 查看 / 收藏带数字，分享无数字），数字用独立 `id="trCount*"` |
| 样式 | `.tr-actions` / `.tr-action` 对齐 `.post-actions` / `.action`（`max-width: 460px`、`gap: 6px`、13px 字号、按类型悬停色），新增 `.tr-count { font-variant-numeric: tabular-nums; }` |
| 同步 | 新增 10.8 段：在现有 5 个数字输入框上**叠加 `input` 监听**，读 `.value || '0'` 写入翻译卡；不改 `render()` 一行 |
| 随机 / 恢复示例 | 这两个动作直接写 `.value`、不触发 `input`，故再叠 `click` + `setTimeout(..., 0)` 排在原监听器之后读新值 |

- 改动位置：`index.html` 的 12.3 CSS 段、翻译卡 `<footer>`、10.8~10.9 JS 段 —— 全部位于 v1.1.0 新增区，**v1.0.1 的 DOM / CSS / JS / 导出一行未动**。

### 验证

- 新增 `verify_counts.js` 专项 **9/9 PASS**：结构 6+5、初始一致、5 个输入实时同步、空值兜底 0、「随机」跟随、「恢复示例」跟随、双形态一致、等宽数字与 6px 间距、无报错
- 本地回归：`verify_v110.js` **18/18 PASS**、`verify_mask.js` **9/9 PASS**
- **导出零变化铁证**：v1.0.1 / v1.1.0 / v1.1.1 同参数 PNG 均 `1324×598`，像素差 **0**，sha1 同为 `453d8eda3e9154dfa38b400a53018aa6f8f8b37b`
- CSS 规则 158 → 162 条（新增 4 条 `tr-*` 规则），v1.0.1 的 122 条规则仍原样保留
- 线上：GitHub Pages / here.now 各跑 `verify_counts.js` + `verify_online_tr.js` + `verify_mask.js`

### 已知既有项（未处理）

同 v1.1.0：`v1.0.1` 在 375 竖屏本就有 5px 横向溢出（老容器），按「现有不要动」约束未修；翻译卡仍不进入导出 PNG。

### 改动文件

| 文件 | 变化 |
| --- | --- |
| `index.html` | 97,591 → 100,162 bytes；12.3 CSS 段、翻译卡 footer、10.8/10.9 JS 段 |
| `README.md` | 功能说明 / 技术实现 / 版本行同步到 v1.1.1 |
| `教案.md` | 第 7 课补 1 条 + 常见坑速查表补 1 行 |
| `RELEASE_NOTES.md` | 本节 |

### 在线地址

- GitHub Pages：https://openclawzhangchong.github.io/x-post-teaching/
- here.now：https://frosty-lichen-w688.here.now/

### 回滚

`git checkout v1.1.1`（annotated tag）；回到修复前用 `git checkout v1.1.0`

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
