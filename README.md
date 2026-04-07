# 个人博客（Astro）

这是一个基于 Astro + TailwindCSS 的个人博客/作品集站点（从 Astro Milidev 模板 fork 后做了精简）。

## 本地开发

```bash
npm ci
npm run dev
```

构建与预览：

```bash
npm run build
npm run preview
```

## 写内容

内容使用 Astro Content Collections，目录在 `src/content/`：

- 博客：`src/content/blog/<slug>/index.md` 或 `index.mdx`
- 项目：`src/content/projects/<slug>/index.md` 或 `index.mdx`
- 分享/演讲：`src/content/talks/<slug>/index.md` 或 `index.mdx`

Frontmatter 字段可参考 `src/content/config.ts`。

## 部署到 GitHub Pages

仓库已内置 GitHub Actions 工作流：`.github/workflows/deploy.yml`，推送到 `main` 分支会自动构建并部署。

站点地址相关配置在 `astro.config.mjs`：

- 默认（Project Pages 未绑定自定义域名）：`site=https://<username>.github.io`，`base=/astro-milidev/`
- 绑定自定义域名后：`site=https://blog.sxshenxue.top`，并移除 `base`（或保持为默认 `/`）
