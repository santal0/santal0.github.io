# GitHub Pages 部署

主页使用 MkDocs 构建，工作流位于 [`workflows/ci.yml`](workflows/ci.yml)。

首次启用：

1. 在 GitHub 仓库的 **Settings → Pages → Build and deployment** 中，将 **Source** 设置为 **GitHub Actions**。
2. 将工作流提交并推送到 `master` 分支。
3. 在 **Actions → Build and deploy MkDocs to GitHub Pages** 查看构建和部署结果，也可以通过 **Run workflow** 选择 `master` 手动运行。

向 `master` 推送会构建并部署；以 `master` 为目标的 PR 只检查构建。手动运行其他分支也只检查构建。部署使用工作流自带的 `GITHUB_TOKEN`，无需额外配置个人访问令牌。若 `github-pages` 环境配置了分支限制，需允许 `master` 部署。

构建成功后上传 `site/`，再由官方 Pages Action 发布到 <https://santal0.github.io/>。工作流不再向 `gh-pages` 分支推送文件。

本地验证（Python 3.11）：

```bash
python -m pip install -r requirements.txt
python -m mkdocs build --strict
```

严格模式下 MkDocs 警告会导致构建失败；请按 Actions 构建日志修复对应的配置、链接或资源问题。

参考：[GitHub Pages 自定义工作流文档](https://docs.github.com/zh/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages)。
