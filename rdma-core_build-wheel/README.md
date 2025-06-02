# 1. 下载 rdma-core

```bash
git clone https://github.com/linux-rdma/rdma-core.git
git checkout v39.0
```

# 2. 安装依赖
```bash
sudo apt-get update
sudo apt-get install -y git make gcc g++ pkg-config libtool autoconf automake \
    python3 python3-pip python3-setuptools python3-wheel libmnl-dev \
    libibverbs-dev libibumad-dev librdmacm-dev libudev-dev libnl-3-dev \
    libnl-route-3-dev libssl-dev libxml2-dev xsltproc docbook-xsl \
    libxml2-utils
```

# 3 编译
```bash
cd rdma-core
bash build.sh
```

# 4. wheel 打包

## 4.1 复制预编译文件
```bash
cd rdma-core
mkdir pyverbs_wheel
cd pyverbs_wheel
mkdir -p src/pyverbs

cp ../build/python/pyverbs/*.so src/pyverbs/
cp -r ../pyverbs/* src/pyverbs/
```

## 4.2 setup.py 文件：

```python
# cd rdma-core/pyverbs_wheel
# setup.py
import os
import sys
from setuptools import setup, find_packages
from setuptools.command.build_ext import build_ext

# 获取当前目录下的.so文件
so_files = [f for f in os.listdir("src/pyverbs") if f.endswith('.so')]

class CustomBuild(build_ext):
    def run(self):
        # 无需编译，直接使用预编译文件
        pass

setup(
    name="pyverbs",
    version="0.1.0",
    package_dir={"": "src"},
    packages=find_packages(where="src"),
    package_data={"pyverbs": so_files},
    cmdclass={"build_ext": CustomBuild},
    install_requires=["cffi"],
    zip_safe=False,
)
```

## 4.3 打包 wheel
```bash
pip install build       # 安装 build 工具
python -m build --wheel # 打包 wheel
```
## 4.4 验证wheel
```bash
# 生成的wheel在 dist/ 目录
ls dist/*.whl

# 测试安装
pip install dist/pyverbs-0.1.0-py3-none-any.whl --force-reinstall
python -c "import pyverbs; print(pyverbs.Context(name='rocep0s8f0'))"
```
