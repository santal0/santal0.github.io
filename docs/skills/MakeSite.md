# 记录如何制作个人网站

**采用 mkdocs 搭建个人网站**

!!! note 
    "本文适用于 Windows 系统的 wsl2 环境"
    使用了 vscode 编辑器，并安装了 remote-ssh 插件，可以远程连接到 wsl2 环境进行操作。
    你需要学会如何使用 Github 进行版本控制，以及如何使用 Markdown 进行文档编写。


## 安装工具链

```shell
pip install mkdocs-material
pip install mkdocs
pip install mkdocs-heti-plugin
pip install mkdocs-statistics-plugin
```

## 部署网站

1. 在 Github 创建一个仓库，命名为 yourusername.github.io，其中 yourusername 是你的 Github 账号。
2. 克隆仓库到本地：

```shell
git clone https://github.com/yourusername/yourusername.github.io
```
3. 进入仓库目录：

```shell
cd yourusername.github.io
```
4. 初始化 mkdocs：

```shell
mkdocs new .
```
5. 配置 mkdocs.yml：
最简单的配置：

```yaml
site_name: 你想给网页取的名字
site_url: https://yourusername.github.io/
theme:
  name: material
```
6. GitHub action 自动部署：
在仓库根目录下创建一个 `.github/workflows/ci.yml` 文件，内容如下：
```yaml
name: ci
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV
      - uses: actions/cache@v3
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install -r requirements.txt
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```
7. 本地预览网站：

```shell
mkdocs build
mkdocs serve
```

8. 上传网站

9. 线上部署
在仓库的 Setting->Pages 页面，选择 gh-pages /root，点击 save，保存。
检查 action 状态，等待部署完成。

10. 访问网站：https://yourusername.github.io/

## 使用卡片
引用自 [shrike505 的友站](https://nest.shrike505.cc/friends/)
```shell
pip install neoteroi-mkdocs
```
官方文档讲的不是很详细，需要把他仓库里的 `scss` 文件拉到本地用 `sass` 编译成 `css` 再引用
可以到我的仓库里下载编译好的 `css/cards.css` 文件，再在 `mkdocs.yml` 里引用：
```yaml
markdown_extensions:
  - neoteroi.cards
extra_css:
  - css/cards.css
```

