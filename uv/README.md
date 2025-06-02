# 1 install

```sh
# python -m pip install --upgrade pip
pip install uv -i https://pypi.tuna.tsinghua.edu.cn/simple
```

# 2 hello uv

```sh
hello_uv/
├── hello/          # 包目录
│   ├── __init__.py
│   └── test.py
├── pyproject.toml
└── README.md
```

- pyproject.toml

```sh
[build-system]
requires = ["setuptools>=65", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "hello-uv"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.10"
dependencies = ["numpy>=1.26"]

[tool.uv]
index-url = "https://pypi.tuna.tsinghua.edu.cn/simple"

[tool.setuptools]
packages = ["hello"]
```

```python
# test.py
def main():
    print("Hello from hello-uv!")


if __name__ == "__main__":
    main()
```

-- uv build 即可完成项目whl构建

```shell
>>> from hello.test import main
>>> main()
Hello from hello-uv!
```

## 2.1 init project
```bash
# 创建并初始化项目
uv init hello_uv
```

此时会自动创建 hello_uv 目录，并在该目录下生成特定配置文件:

```sh
.  ..  .python-version  README.md  main.py  pyproject.toml
```

> main.py 何 pyproject.toml 会有默认内容，可以根据需要修改。

## 2.2 创建虚拟环境

```bash
cd hello_uv && uv sync

# method 2
uv venv .venv  # 创建虚拟环境到 .venv 目录（default .venv）
```

> 此时会在hello_uv目录下生成一个.venv目录，里面包含python虚拟环境。


- 往虚拟环境中安装依赖包

```bash
uv add numpy # 添加依赖, 同时修改pyproject.toml文件和.venv

uv remove requests  # 移除依赖，同时修改pyproject.toml 和.venv

# --dev : 开发依赖，生产不依赖
uv add --dev black    # 添加开发依赖（代码格式化）

uv remove --dev flake8  # 移除开发依赖

uv tree                 # 显示依赖树
```

**uv pip install : 利用外面的pip 来单纯的往uv 的虚拟环境安装依赖包, 不会修改pyproject.toml**

**注意** <br>
> 1. uv add numpy 会解析命令并更新 pyproject.toml 中的依赖列表。
> 2. 然后 uv 会调用内部集成的 pip（或系统环境中的 pip）来下载并安装 numpy 到当前项目的虚拟环境中（.venv）。
> 3. 安装完成后，uv 还会处理依赖关系，并确保环境一致性。
> 4. 也可用通过uv pip install numpy 安装依赖到uv 管理的环境中，且不修改pyproject.toml.

## 2.3 运行项目
```bash
uv run main.py # 不需要写python
```

## 2.4 构建项目

- uv build：生成可分发包（**用于发布或部署**）
- uv pip install：安装当前项目到本地环境;

```bash
# 该命令会执行项目的构建流程，并触发 .egg-info 的生成
uv build # 构建项目
```

> 此命令会生成 可分发的包文件（如 .whl 轮式包或 .tar.gz 源码包），输出到 dist/ 目录。

如果你需要更细粒度控制，可以直接使用 setuptools：

```bash
python setup.py egg_info
```

> 注意：注意：你需要确保 setup.py 存在（如果项目是基于 setuptools 构建系统）。

**是否会用setup.py 来构建** <br>

在 pyproject.toml 中配置 build-backend = "setuptools.build_meta" 时，构建行为取决于项目中是否存在 setup.py 文件：<br>

|配置	|存在 setup.py	|无 setup.py|
|-------|------------------|--------------|
|build-backend = "setuptools.build_meta"	| ✅ 执行 setup.py	| ✅ 使用 [project] 配置|
|build-backend = "setuptools.build_meta:__legacy__" |	✅ 执行 setup.py	| ❌ 失败 |


## 2.4 生成的文件

- uv.lock

> 作用：锁定项目依赖的精确版本，确保在不同环境中安装的依赖完全一致。
> 内容：包含所有依赖及其子依赖的具体版本号、哈希值和来源信息。
> 用途：用于可重复构建（reproducible builds），防止因依赖版本变动导致的行为差异。
> 生成时机：当运行 **uv sync 或 uv add 等命令时自动生成或更新**。

- .python-version

> 作用：指定项目使用的 Python 版本。
> 内容：例如 3.10，表示该项目期望使用 Python 3.10。
> 用途：被工具链（如 pyenv 或 uv）用来自动选择正确的 Python 解释器。
> 生成时机：运行 uv init 时根据当前环境自动生成特定版本(建立软链接)，不过我们可用修改该文件来指定特定版本。

- hello_uv.egg-info

> 作用：存放项目的元数据（metadata），用于构建和分发。
> 内容：
> - PKG-INFO: 包的基本信息（如名称、版本、依赖等
> - SOURCES.txt: 包含哪些源代码文件
> - dependency_links.txt: 依赖链接
> - top_level.txt: 主模块名
> 用途：主要用于兼容 setuptools 构建系统，在安装或打包时被用到。
> 生成时机：**首次**运行 uv sync 或执行构建操作时生成。

---

# 3 uv 常用指令
以下是 **`uv` 包管理工具最新常用功能的精简总结（截至2025年6月）**，聚焦高效开发工作流：

## 3.1 **🔥 核心工作流（5大高频命令）**
1. **`uv venv`**
   - 创建虚拟环境：`uv venv`（默认 `.venv`）
   - 指定 Python 版本：`uv venv --python=3.12`

2. **`uv add`**
   - 安装主依赖：`uv add requests pandas`
   - 安装开发依赖：`uv add -D pytest ruff`
   - 升级包：`uv add --upgrade pandas`

3. **`uv remove`**
   - 卸载包：`uv remove requests`

4. **`uv sync`**
   - 同步依赖：`uv sync`（自动读取 `pyproject.toml` + 锁文件）
   - 指定文件：`uv sync --requirements-file=requirements-dev.txt`

5. **`uv run`**
   - 运行命令：`uv run pytest`
   - 启动应用：`uv run python main.py`

---

## 3.2 **⚡️ 进阶技巧**
| **场景**               | **命令**                                      |
|------------------------|-----------------------------------------------|
| **初始化新项目**       | `uv venv && uv add .`                         |
| **批量升级依赖**       | `uv add --upgrade-all`                        |
| **生成精确锁文件**     | `uv pip compile pyproject.toml -o requirements.txt` |
| **跨平台环境同步**     | `uv sync --python=3.11`                      |
| **临时执行命令**       | `uv run --command "python -c 'import sys; print(sys.path)'"` |

---

### **🚀 性能优化**
```bash
# 使用清华镜像源（提速 5X+）
export UV_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple

# 并行安装（默认启用）
uv pip install -r requirements.txt --parallel

# 跳过缓存验证（极端提速）
uv pip install --no-cache
```

---

## 3.3 **💡 智能特性**
1. **自动依赖追踪**
   - `uv add` 自动更新 `pyproject.toml` 依赖声明
   - `uv remove` 自动移除依赖声明

2. **环境自适配**
   - `uv run` 自动检测当前目录的 `.venv`
   - 未激活环境时直接执行命令（无需手动 `activate`）

3. **锁文件生成**
   ```bash
   # 生成跨平台兼容的锁文件
   uv pip compile --generate-hashes -o requirements.txt
   ```

---

## 3.4 **🚨 注意兼容性**
- 快捷命令需 `uv >= v0.1.7`（2024年5月+）
- 升级命令：
  ```bash
  # 通过 pipx 升级
  pipx upgrade uv

  # 或直接升级
  python -m pip install --upgrade uv
  ```

## 3.5 **典型工作流**：
> ```bash
> uv venv                          # 创建环境
> uv add fastapi uvicorn           # 安装核心依赖
> uv add -D pytest                 # 安装测试工具
> uv run uvicorn app:app --reload  # 启动开发服务器
> ```

# 参考
- [](https://docs.astral.sh/uv/guides/configuration/)