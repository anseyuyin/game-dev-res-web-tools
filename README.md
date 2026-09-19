# 🎮 游戏开发资源工具集（game-dev-res-web-tools）

面向游戏开发者的**单文件 Web 工具集合**。所有工具都是独立的 `.html` 文件，双击即可在浏览器中运行，无需安装、无需构建、无需后端服务，处理过程全部在本地完成。

> 在线索引页：[`tools/tools_index.html`](tools/tools_index.html)

---

## ✨ 特性

| 特性 | 说明 |
| --- | --- |
| 🗂️ 单一 HTML | 一个工具就是一个 `.html` 文件，复制即可用、即可分享 |
| 🔍 统一索引 | [`tools/tools_index.html`](tools/tools_index.html) 提供全部工具的展示、搜索与收藏筛选 |
| ⭐ 收藏筛选 | 收藏状态保存在浏览器 `localStorage`，支持「只看收藏」 |
| 🏷️ 标签检索 | 支持按名称、描述、标签、文件名搜索 |
| 🔒 本地运行 | 不依赖 CDN、不上传数据，断网也能用 |
| 📦 零依赖 | 不需要 Node.js / Python / 任何运行时环境 |

---

## 🚀 快速开始

### 使用工具

1. 用浏览器打开 [`tools/tools_index.html`](tools/tools_index.html)（也可以直接双击文件）。
2. 在搜索框中输入关键词，或用标签 / ⭐ 收藏筛选定位工具。
3. 点击卡片进入对应工具页面开始使用。

> 快捷键：按 `/` 聚焦搜索框，按 `Esc` 清空搜索。

### 本地浏览（可选）

如果希望用 `http://` 方式访问（更方便测试下载、剪贴板等 API），可在仓库根目录执行：

```powershell
# 任意一种均可
python -m http.server 8080
# 或
npx serve .
```

然后访问 `http://localhost:8080/tools/tools_index.html`。

---

## 📁 目录结构

```
game-dev-res-web-tools/
├─ README.md                     # 项目总览（本文件）
├─ LICENSE                       # 开源协议
├─ .gitignore
├─ docs/                         # 中文文档
│  ├─ 工具索引.md                 # 全部工具清单与说明
│  └─ 开发规范.md                 # 新增工具的开发规范与步骤
└─ tools/                        # 所有工具（每个工具一个 .html）
   ├─ tools_index.html           # 工具索引页（搜索 + 收藏筛选）
   └─ <your-tool>.html           # 你的工具（以 _ 开头的文件不登记进索引）
```

---

## 🧩 现有工具

工具清单维护在 [docs/工具索引.md](docs/工具索引.md) 中，与索引页的 `TOOLS` 数组保持一致。

当前工具数量：**1**

| 工具 | 说明 | 标签 |
| --- | --- | --- |
| 🎬 [绿幕动作视频 → 序列帧图集](tools/video_to_sequence_frame_tool.html) | 把游戏单位的绿幕动作视频抽帧、抠像、裁剪，导出带 Alpha 的序列帧 PNG 或单张图集，并可在页面内试播动作 | `视频` `序列帧` `抠像` `图集` |

详细用法、工作流与限制见 [docs/工具索引.md](docs/工具索引.md)。

---

## ➕ 新增一个工具

1. 在 `tools/` 下新建语义化的文件，例如 `tools/texture-atlas-packer.html`；样式与交互可参考已有的 `tools/video_to_sequence_frame_tool.html`。
2. 在文件内实现你的功能：CSS / JS / 图标全部内联，不要引用外部资源。
3. 打开 `tools/tools_index.html`，在脚本顶部的 `TOOLS` 数组中追加一条记录：

```js
{
  id:   "texture-atlas-packer",          // 唯一标识，用于收藏存储，勿随意改动
  name: "纹理图集打包器",                  // 工具名称
  desc: "把多张小图合并成一张图集并输出坐标 JSON。", // 一句话描述
  file: "texture-atlas-packer.html",     // 相对 tools_index.html 的路径
  icon: "🧵",                            // 一个 emoji
  tags: ["美术", "纹理", "打包"]           // 标签，用于筛选
}
```

4. 在 [docs/工具索引.md](docs/工具索引.md) 的清单表格与详情小节中补上这个工具。
5. 提交：`git add . && git commit -m "feat: 新增纹理图集打包器"`。

详细规范见 [docs/开发规范.md](docs/开发规范.md)。

---

## 📐 项目原则

1. **单一文件**：每个工具就是一个 `.html`，自包含、可离线、可单独分享。
2. **零依赖**：不引入外部库、CDN、构建工具；不产生 `node_modules`。
3. **本地优先**：不上传、不联网、不追踪，用户数据只留在本机。
4. **索引统一**：[`tools/tools_index.html`](tools/tools_index.html) 是所有工具的唯一入口，新增工具必须登记。
5. **命名清晰**：新文件名与 `id` 使用 kebab-case，例如 `sprite-sheet-slicer.html`（早期已有的 `video_to_sequence_frame_tool.html` 保留原名，见[开发规范](docs/开发规范.md)）。

---

## 🤝 贡献

欢迎提交新的游戏开发资源工具。请先阅读 [docs/开发规范.md](docs/开发规范.md)，确保符合单一文件、零依赖、本地运行的要求。

---

## 📄 许可证

本项目基于 [LICENSE](LICENSE) 中声明的协议开源。
