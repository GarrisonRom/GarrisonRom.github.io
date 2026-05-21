# GarrisonRom Tech Site

一套完整的个人技术博客站点，包含主页、文章、技术文档中心、以及带权限的管理后台。

## 文件结构

```
garrisonrom-site/
├── index.html              # 主页（中英文双语）
├── style.css               # 全局样式（GitHub 深色主题）
├── posts/                  # 文章详情页
│   ├── continual-edge-design.html
│   ├── why-llm-post-training.html
│   ├── elevator-saas-30days.html
│   ├── iclr-domain-fingerprint.html
│   └── neuro-loop-bci.html
├── docs/                   # 技术文档中心
│   ├── index.html          # 文档中心入口
│   ├── projects.html       # 项目文档
│   ├── papers.html         # 论文档案
│   ├── tech-spec.html      # 技术规范（ContinualEdge）
│   ├── research-direction.html  # 研究方向
│   └── tech-stack.html     # 技术栈
└── admin/                  # 管理后台
    ├── login.html          # 登录页
    └── dashboard.html      # 内容管理面板
```

## 使用方式

### 1. 本地预览
直接用浏览器打开 `index.html` 即可。所有页面均为纯静态 HTML，无需服务器。

### 2. 部署到 GitHub Pages
1. 创建 GitHub 仓库（如 `GarrisonRom.github.io`）
2. 将这些文件推送到仓库根目录
3. 开启 GitHub Pages（Settings → Pages → Source: main branch）
4. 访问 `https://garrisonrom.github.io`

### 3. 管理后台（Admin）
- 访问 `admin/login.html`
- 默认账号：`garrison`
- 默认密码：`scut2026`（建议修改）
- 登录后可编辑：学习心得、研究方向、技术栈、项目状态、论文追踪、站点配置
- 所有数据保存在浏览器 localStorage 中，支持导出/导入 JSON 备份

**注意**：当前登录验证为前端模拟，仅适用于个人本地使用。若需线上部署，请增加后端鉴权。

## 技术特点

- **纯静态**：无构建工具，无依赖，直接可运行
- **双语切换**：中英文一键切换（JavaScript 控制 CSS 显隐）
- **响应式**：适配手机端（2.4 寸屏到桌面端）
- **深色主题**：GitHub 风格配色，护眼且专业
- **模块化**：新增文章只需在 `posts/` 下新建 HTML，主页自动链接

## 自定义指南

### 修改个人信息
编辑 `admin/dashboard.html` 中的 `DEFAULTS.settings`，或登录后台修改站点配置。

### 新增文章
1. 在 `posts/` 下新建 `.html` 文件（复制现有文章模板）
2. 在 `index.html` 的 `#posts` 区块添加链接
3. 在 admin 后台的"学习心得"面板添加条目

### 修改密码
编辑 `admin/login.html` 中的 `ADMIN_PASS` 常量。

## 配色参考

| Token | 色值 | 用途 |
|-------|------|------|
| `--bg` | `#0d1117` | 主背景 |
| `--bg-secondary` | `#161b22` | 卡片背景 |
| `--border` | `#30363d` | 边框 |
| `--text` | `#c9d1d9` | 主文字 |
| `--text-heading` | `#f0f6fc` | 标题 |
| `--blue` | `#58a6ff` | 强调色/链接 |
| `--green` | `#3fb950` | 成功/录取 |
| `--orange` | `#f0883e` | 警告/投稿中 |
| `--red` | `#f85149` | 错误/拒绝 |

---
Built by Jiaxin Luo (GarrisonRom).
