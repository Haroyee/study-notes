# Python

# 一、安装

## 1、使用uv管理python项目

下载uv(需要魔法)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

更新uv

```bash
uv self update
```

查看支持的python版本

```bash
uv python list
```

下载指定版本的python

```bash
uv python install 3.x.x
```

使用指定虚拟环境的python

```bash
uv venv --python 3.x.x
source .venv/bin/activate
```

2、修改uv的下载镜像

配置文件路径

| 操作系统    | 配置文件路径             |
| ----------- | ------------------------ |
| **Linux**   | `~/.config/uv/uv.toml`   |
| **macOS**   | `~/.config/uv/uv.toml`   |
| **Windows** | `%APPDATA%\\uv\\uv.toml` |

在配置文件路径修改或者新建`uv.toml`

```toml
[pip]
index-url = "https://pypi.tuna.tsinghua.edu.cn/simple"
```

3、uv创建python项目

下载依赖（要有**`pyproject.toml`**文件）

```bash
uv sync
```

