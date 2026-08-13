# dean.github.io

Dean 的个人博客与测试笔记仓库。

## 目录结构

- `myblogs/` — Hexo 博客源码
  - `source/_posts/` — 博客文章
  - `public/` — 本地构建产物（不入库，由 CI 生成并发布到 `gh-pages`）
- `测试笔记/`、`Ai 测试/`、`测试工具/`、`课外拓展/` — Obsidian 笔记库

## 博客构建与部署

本地预览（在 `myblogs/` 下）：

```bash
npm install
npm run server
```

构建静态站：`npm run build`，产物输出到 `myblogs/public/`。

推送 `main` 后，GitHub Actions（`.github/workflows/deploy-blog.yml`）会自动在 `myblogs/` 执行 `npm ci && npm run build`，并把 `public/` 发布到 `gh-pages` 分支；GitHub Pages 站点由 `gh-pages` 分支提供。

首次启用：

1. 推送 `main` 分支（触发一次 Actions 部署）。
2. 在仓库 Settings → Pages 中把 Source 选为 `Deploy from a branch` → `gh-pages` / `(root)`。
3. 站点地址为 <https://17867940894.github.io/>。
