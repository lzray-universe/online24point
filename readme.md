
# 在线 24 点

一个单文件的在线 24 点小游戏，支持**单机**与**多人**两种模式，并内置一套**难度评估与 par（参考完成时长）**算法。仓库自带 GitHub Actions。

---

## 玩法规则（24 点简述）

* 使用四张牌（本项目用数字 `1–13` 表示 A–K），**每张牌必须且只能用一次**。
* 允许的运算：`+  -  ×  ÷`，可自由加括号改变优先级。
* 目标：通过合法运算得到结果 **24**。
* 不允许其它高阶运算（如乘方、阶乘等）。
* 判题以**确切数值**为准（项目内部用分数/浮点混合校验，界面容差 `1e-6`）。

---

## 本项目的特点

### 双模式：

* **单机冲刺（10 题）**

  * 随机生成「**保证可解**」的题面；
  * 内置 **par**（参考完成时长）；
  * 提供 **撤销 / 清空 / 看一个可行式子** 按钮；
  * 实时计时、统计胜场、总用时与 par 总和。

* **多人对战**

  * 输入 **房间号** 与 **昵称** 即可加入；
  * 一键 **创建房间**、**主持人开赛/重置**、**全员同意进入下一轮**；
  * 展示玩家列表、连接状态、排名与本轮进度；
  * 采用 **WebSocket**（Cloudflare Workers 后端）进行同步。

### 难度评估 + par 计算

* 用**严格的分数运算和穷举搜索**评估题面难度：

  * 运算基础权重：`+`、`-`、`×`、`÷` 难度不同；
  * **整除奖励**、**分数惩罚**；
  * **负数/大中间数惩罚**、**深度惩罚**；
  * **特殊数字奖励**：中间产出 `12/8/6/4/3/2` 等容易联想到 24 的中间量更容易；
  * **情境配对奖励**：如得到 `12` 且剩余数字包含 `2`，更利于最终凑 `24`。
* 将综合难度 `x` 映射为 **par（秒）**：

  ```
  par=round(13+log2(x+3)*10)
  ```

  每题的计时与 par 对比，可反馈“超常/一般/偏慢”。

###  UI / UX

* **暗黑 / 明亮**一键切换；
* 响应式布局、圆角；
* **零依赖**：单个 `index.html` 即可离线运行/部署。

---

## 仓库结构

```
.
├── index.html                  # 单页应用（UI、逻辑、难度/搜索/求解）
├── assets/
│   └── favicon.svg
└── .github/
    └── workflows/
        └── deploy.yml          # GitHub Pages 自动部署工作流
```

---

## 快速开始

### 本地

1. 克隆或下载本仓库；
2. 双击打开 `index.html` 即可。

### 单机模式

* 点击「开始 10 题」→ 卡片或键盘输入 → 括号与 `+ - × ÷` 组合；
* 「提交当前题」进行校验；
* 「看一个可行式子」获得参考表达式；
* 用时 **≤ par** 记为 **胜**。

### 多人模式

1. 输入 **房间号**（如 `5D4K`）与**昵称**；
2. **创建房间**（成为主持人）或 **加入房间**；
3. 主持人点击「开始发题」，对局开始；
4. 任一题结束后，可「同意开始下一轮」，由主持人「重置开新一轮」。

> **提示**：默认后端为 `https://24worker.lzraylzraylzray.workers.dev`。若你 Fork 自用，请在 `index.html` 中修改：
>
> ```js
> const WORKER_BASE = "https://你的Workers域名";
> const WS_BASE = WORKER_BASE.replace(/^http/i, "ws");
> ```

---

## CI/CD：

本仓库已包含以下工作流（`.github/workflows/deploy.yml`），当你向 `main` 分支推送时，会自动将站点发布到 **GitHub Pages**。

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [ main ]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v5
        with:
          enablement: true
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .
      - name: Deploy
        uses: actions/deploy-pages@v4
```

### 一次性设置

1. 在 GitHub 仓库 **Settings → Pages** 中，将 **Build and deployment** 设置为 **GitHub Actions**；
2. 首次运行后，Pages 将生成访问地址（`https://<owner>.github.io/<repo>`）。

> 工作流无需任何私密令牌或额外配置；本项目为纯静态站点，直接上传生成物（仓库根目录）到 Pages。

---

## 自定义与高级用法

* **更换后端**（多人对战）：在 `index.html` 顶部修改 `WORKER_BASE`；
* **调整难度口味**：搜索常量 `const W = { ... }` 可微调各类权重（整除奖励、分数惩罚、深度惩罚、“钩子数”与情境配对奖励等），par 会随之变化；
* **改主题**：CSS 变量集中在 `<style>` 的 `:root` 与 `body[data-theme]` 中。

