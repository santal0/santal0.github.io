# santal0.github.io

## 部署说明

* 克隆仓库

```shell
git clone git@github.com:santal0/santal0.github.io.git
cd santal0.github.io
```

* 配置环境

```shell
conda create -n mypage python=3.11
conda activate mypage
python -m pip install -r requirements.txt
```

* 预览网页

```shell
python -m mkdocs serve
```

* 部署前检查（与 GitHub Actions 一致）

```shell
python -m mkdocs build --strict
```
