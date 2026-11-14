# 项目阶段性交接文档 (HANDOVER.md)

> **归档时间**：2026-11-14  
> **项目名称**：雨夜长江航道趸船 · 3D安全隐患排查小游戏 & 三渲二微缩景观  
> **当前状态**：第一阶段（核心场景拟真、全量隐患开发、双端自适应、独立部署架构）已全面成熟并完成线上验证。后续迭代直接基于本基线展开。

---

## 1. 已拍板共识

### 1.1 业务场景与船舶原型
- **船舶原型**：长江航道局典型综合性趸船，正式定名 **“汉道趸09408”**（原命名“武道趸30402”已作废）。
- **视觉风格**：雨夜沉浸式三渲二卡通微缩风格（Cel-Shading 结合描边轮廓），包含雨滴涟漪水面 Shader、夜间灯光与环境雾效。
- **内河航道规范**：
  - 顶板红绿舷灯：严格遵守规范，红灯位于趸船背面左舷，绿灯位于正面右舷，带遮光板与警示光源。
  - 官方 Logo：船顶中央鼓形灯箱双面贴装长江航道局官方圆形徽标（CHANGJIANG WATERWAY）。
  - 标牌与警示：包含“安全逃生通道”、“太平斧”、“安全生产信息公示”及黄黑斑马反光警戒线等。

### 1.2 架构与网络选型（0 外部网络依赖）
- **纯前端自包含架构（Zero External Network Dependency）**：
  - 将 `Three.js r128` (~600KB) 与 `OrbitControls` (~26KB) 直接内嵌在 HTML 内。
  - 长江航道局官方 Logo 转换无损 Base64 Data URI 内嵌在脚本中。
  - 彻底规避外部 CDN 在国内访问受阻、超时以及 Cloudflare SPA 404 MIME 类型冲突导致引擎打不开的问题。
- **双重页面路由结构**：
  - **微缩景观首页**：根路径 `/` (`index.html`)，纯享展示雨夜长江航道趸船三渲二微缩景观，右上角挂载轻量毛玻璃胶囊链接 `🎮 安全隐患排查游戏 →`。
  - **隐患排查游戏页**：`/game`, `/game.html`, `/safegame2.html`，自适应桌面端与移动端排查小游戏。
- **部署与发布标准**：
  - **Cloudflare Pages**：项目名称 `yu-ye-changjiang`，域名 `https://yu-ye-changjiang.pages.dev/`。采用 Direct Upload API（BLAKE3-32 哈希与 `upsert-hashes` 机制）。
  - **GitHub 仓库**：`https://github.com/BIBLEofDark/SafeGame.git`（分支 `main`），通过 GitHub API 自动同步最新源码。

### 1.3 游戏机制与隐患规则
- **题库规模**：共 19 处典型生产生活安全隐患（编号 `H01`~`H10`, `H12`~`H20`）。
- **游戏时限（已压缩至原 1/2）**：
  - 简单模式：抽查 6 个隐患，限时 2.5 分钟，带橙色脉冲球提示。
  - 普通模式：抽查 8 个隐患，限时 1 分 45 秒，带橙色脉冲球提示。
  - 困难模式：抽查 10 个隐患，限时 1 分 15 秒，无视觉标记。

---

## 2. 当前进度与产物

### 2.1 核心文件列表
- 🎮 [safegame2.html](file:///f:/SafeGame/safegame2.html)：安全隐患排查游戏单文件版（带内嵌引擎、19 项题库、移动端紧凑菜单与背面运镜）。
- 🚢 [index.html](file:///f:/SafeGame/index.html)：微缩景观展示页（带内嵌引擎、官方 Logo 与游戏入口悬浮胶囊）。
- 📂 [game/index.html](file:///f:/SafeGame/game/index.html)：对齐 `/game` 路由的游戏入口文件。
- 📦 [dist/](file:///f:/SafeGame/dist)：Cloudflare Pages 部署包目录（含 `index.html`, `game.html`, `safegame2.html`, `game/index.html`, `game/safegame2.html`）。
- 📖 [README.md](file:///f:/SafeGame/README.md)：项目结构说明、特性与隐患明细手册。

### 2.2 线上端点当前状态
| 端点 | 对应文件 | 当前内容 | 验证状态 |
| :--- | :--- | :--- | :--- |
| `https://yu-ye-changjiang.pages.dev/` | `index.html` | 雨夜长江航道趸船 · 三渲二微缩景观 | HTTP 200 正常 |
| `https://yu-ye-changjiang.pages.dev/game` | `game.html` / `game/index.html` | 雨夜长江航道趸船 · 安全隐患排查小游戏 | HTTP 200 正常 |
| `https://yu-ye-changjiang.pages.dev/safegame2.html` | `safegame2.html` | 雨夜长江航道趸船 · 安全隐患排查小游戏 | HTTP 200 正常 |

---

## 3. 已排除路径与教训总结

1. **排除外部公共 CDN 引用**：
   - 曾通过 `cdnjs.cloudflare.com` / `cdn.jsdelivr.net` 加载引擎，国内网络极不稳定，且 Cloudflare Pages 在请求不存在资源时回退 SPA HTML（返回 200 且附带 `nosniff`），导致浏览器报错拒绝执行脚本。
   - **定论**：坚持采用内联模式（Zero External Network Dependency）。
2. **排除单页面覆盖方案**：
   - 曾将 `safegame2.html` 覆盖根目录 `index.html`，导致原有的三渲二微缩景观主页被顶替。
   - **定论**：保持 `/`（微缩景观）与 `/game`（排查游戏）分离架构。
3. **排除直觉反三角拼接计算楼梯几何**：
   - 原角度使用补角差值，导致倾斜梯梁（槽钢）与水平踩板在三维空间形成交叉逆向错位。
   - **定论**：梯梁旋转角度严格使用 `Math.atan2(-height, runZ)`，踏步板横向水平嵌入两侧槽钢内部，并补充垂直立柱与顶层扶手。
4. **排除无 AST 静态检查的脚本拼接**：
   - 曾因函数末尾就绪代码块重复黏贴，导致同一个 `try` 块内两次声明 `const spinner`，造成编译期 `SyntaxError` 引起全白/黑屏。
   - **定论**：重大脚本合并必须通过 Esprima AST 语法树扫描，确认 0 语法错误和 0 块级变量重名冲突。

---

## 4. 下一步行动

在新会话开启后，可直接基于本文件开展后续工作：
1. **隐患扩充**：若需继续增加隐患，遵循 `HAZARD_DATABASE` 与 `registerHazardNode` 既有规范在 `safegame2.html` 中增量添加。
2. **答题结果持久化/评分排行**：如未来需要对接后端 API（如记录职工排查用时、生成成绩报告、提交至局安全培训系统），可在结算弹窗处外挂通信模块。
3. **视觉细节进阶**：如需新增舱室内部精细化家具或航道声效（汽笛、雨声环境音），可在现有音效与材质系统中继续挂载。
