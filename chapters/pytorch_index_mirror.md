<h1 align="center">安装 pytorch 包并且指定 cpu 或者 cuda 版本并使用镜像</h1>  

- [https://docs.astral.sh/uv/guides/integration/pytorch/#installing-pytorch](https://docs.astral.sh/uv/guides/integration/pytorch/#installing-pytorch)

ps . uv 官方真的太贴心了，这是喂饭级的，把各种`cpu`,`cu11`,`cu12`,`intel gpu`...的基础配置都写出来了。<br>

我印象里面看到`poetry`用户在安装的时候考虑用指定 whl 路径的方法，但是存在很大局限性，因为 wh l 是锁死 python 版本和系统的。

[Installing a specific PyTorch build (f/e CPU-only) with Poetry](https://stackoverflow.com/questions/59158044/installing-a-specific-pytorch-build-f-e-cpu-only-with-poetry)

## 问题描述：

正常情况下，我们使用 `uv add torch==2.1.0`时，安装的是 cpu+cuda 版本的 torch:(不同操作系统可能有区别)<br>

```shell
xnne@xnne-PC:~/code/Auto_Caption_Generated_Offline$ uv add torch==2.1.0 torchaudio==2.1.0
⠴ nvidia-cusparse-cu12==12.1.0.106         ^C
```

```txt
To start, consider the following (default) configuration, which would be generated by running uv init --python 3.12 followed by uv add torch torchvision.
首先，请考虑以下（默认）配置，运行 uv init --python 3.12 后再运行 uv add torch torchvision 即可生成该配置。

In this case, PyTorch would be installed from PyPI, which hosts CPU-only wheels for Windows and macOS, and GPU-accelerated wheels on Linux (targeting CUDA 12.4):
在这种情况下，可以从 PyPI 安装 PyTorch，PyTorch 在 Windows 和 macOS 上只支持 CPU 驱动轮，在 Linux 上支持 GPU 加速驱动轮（针对 CUDA 12.4）：
```

```toml
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
  "torch>=2.6.0",
  "torchvision>=0.21.0",
]
```

这有时候并不是我们想要的，比如我的 deepin 上面没有装 nvidia 的驱动，那么我多下了 3~4G 的环境我压根用不到。<br>

我尝试用`uv add torch==2.1.0+cpu -f https://download.pytorch.org/whl/torch_stable.html`

```shell
xnne@xnne-PC:~/code/Auto_Caption_Generated_Offline$ uv add torch==2.1.0+cpu -f https://download.pytorch.org/whl/torch_stable.html

Resolved 63 packages in 10.37s
      Built auto-caption-generate-offline @ fil
⠹ Preparing packages... (4/5)
torch      ------------------------------ 19.06 MiB/176.29 MiB
```

它确实下载到了 cpu 版本的 torch,但是，在我尝试从我的远程仓库安装时，我碰到了问题：<br>

```shell
xnne@xnne-PC:~/code/test/Auto_Caption_Generated_Offline$ uv pip install git+https://github.com/MrXnneHang/Auto_Caption_Generated_Offline@v2.4-cpu
    Updated https://github.com/MrXnneHang/Auto_Caption_Generated_Offline (12065e01ec1dc11f8f224fbb132cfd1c18ec3ac1)
  × No solution found when resolving dependencies:
  ╰─▶ Because there is no version of torch==2.1.0+cpu and auto-caption-generate-offline==2.4.0 depends on torch==2.1.0+cpu, we can conclude that auto-caption-generate-offline==2.4.0 cannot be used.
      And because only auto-caption-generate-offline==2.4.0 is available and you require auto-caption-generate-offline, we can conclude that your requirements are unsatisfiable.
```

## 原因：

原因是 pytorch 上传的镜像并不在 pypi 的 index 上。<br>

```txt
From a packaging perspective, PyTorch has a few uncommon characteristics:
从包装的角度来看，PyTorch 有几个不同寻常的特点：

Many PyTorch wheels are hosted on a dedicated index, rather than the Python Package Index (PyPI). As such, installing PyTorch often requires configuring a project to use the PyTorch index.
许多 PyTorch 轮子托管在专用索引上，而不是 Python 包索引 (PyPI)。因此，安装 PyTorch 通常需要将项目配置为使用 PyTorch 索引。

PyTorch produces distinct builds for each accelerator (e.g., CPU-only, CUDA). Since there's no standardized mechanism for specifying these accelerators when publishing or installing, PyTorch encodes them in the local version specifier. As such, PyTorch versions will often look like 2.5.1+cpu, 2.5.1+cu121, etc.
PyTorch 会为每种加速器（如纯 CPU、CUDA）生成不同的编译版本。由于在发布或安装时没有指定这些加速器的标准化机制，PyTorch 将它们编码在本地版本说明符中。因此，PyTorch 版本通常看起来像 2.5.1+cpu , 2.5.1+cu121 等。

Builds for different accelerators are published to different indexes. For example, the +cpu builds are published on https://download.pytorch.org/whl/cpu, while the +cu121 builds are published on https://download.pytorch.org/whl/cu121.
不同加速器的编译会发布到不同的索引中。例如， +cpu 版本发布在 https://download.pytorch.org/whl/cpu 上，而 +cu121 版本发布在 https://download.pytorch.org/whl/cu121 上。
```

## 解决：

最后我自己敲定的相关配置是这样的: [pyproject.toml](https://github.com/MrXnneHang/test-uv-index/blob/master/pyproject.toml)<br>

ps: 如果你是配置 cuda 版本，我们应该考虑使用 previous 版本的 cuda,比如使用 11.8.而不是使用最新的 12.x 甚至 13.x 因为用户的驱动不会一直是最新的，而新的驱动是兼容旧的 cu 版本的。除非在性能上有非常高的提升，但是一般来说是没有太大区别的。<br>

## 总结

**你既可以用 index 来选定 cpu 或者 cuda, 也可以用 index 来使用选择镜像源哦。**<br>

比如用 *https://mirror.nju.edu.cn/pytorch/whl/cu126* 替换 *https://download.pytorch.org/whl/cpu*. 不过, 有时候似乎镜像的安装会[存在问题](https://github.com/astral-sh/uv/issues/12460), 这种时候你稍微升级或者降级所选的 pytorch 版本就可以解决, 不宜用最新, 不宜用太旧, 除非你是框架开发者.<br>

这里是一个参考仓库: [MrXnneHang/test-uv-index](https://github.com/MrXnneHang/test-uv-index)

## 从 github 上安装：

最终尝试从 github 安装 =-=。<br>

```shell
➜  explore-uv git:(tool-uv-index) uv venv -p 3.11.11
Using CPython 3.11.11
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate
➜  explore-uv git:(tool-uv-index) ✗ uv pip install git+https://github.com/MrXnneHang/test-uv-index@master
    Updated https://github.com/MrXnneHang/test-uv-index (fc288a1ed9a5e1bc6ed8fee8cc9ddc42cfd826f7)
Resolved 21 packages in 14.61s
      Built test-uv-index @ git+https://github.com/MrXnneHang/test-uv-index@fc288a1ed9a5e1bc6ed8fee8cc9ddc42cfd826f7
Prepared 6 packages in 2.80s
Installed 21 packages in 195ms
 + contourpy==1.3.2
 + cycler==0.12.1
 + filelock==3.18.0
 + fonttools==4.58.0
 + fsspec==2025.3.2
 + jinja2==3.1.6
 + kiwisolver==1.4.8
 + markupsafe==3.0.2
 + matplotlib==3.10.1
 + mpmath==1.3.0
 + networkx==3.4.2
 + numpy==1.26.4
 + packaging==25.0
 + pillow==11.2.1
 + pyparsing==3.2.3
 + python-dateutil==2.9.0.post0
 + six==1.17.0
 + sympy==1.14.0
 + test-uv-index==1.0.0 (from git+https://github.com/MrXnneHang/test-uv-index@fc288a1ed9a5e1bc6ed8fee8cc9ddc42cfd826f7)
 + torch==2.1.0+cpu
 + typing-extensions==4.13.2
➜  explore-uv git:(tool-uv-index) ✗ uv run test-uv
matplotlib:3.10.1
numpy:1.26.4
torch:2.1.0+cpu
torch cuda available:False
tensor([ 0.0000e+00,  6.3424e-02,  1.2659e-01,  1.8925e-01,  2.5115e-01,
         3.1203e-01,  3.7166e-01,  4.2979e-01,  4.8620e-01,  5.4064e-01,
         5.9291e-01,  6.4279e-01,  6.9008e-01,  7.3459e-01,  7.7615e-01,
         8.1458e-01,  8.4973e-01,  8.8145e-01,  9.0963e-01,  9.3415e-01,
         9.5490e-01,  9.7181e-01,  9.8481e-01,  9.9384e-01,  9.9887e-01,
         9.9987e-01,  9.9685e-01,  9.8982e-01,  9.7880e-01,  9.6384e-01,
         9.4500e-01,  9.2235e-01,  8.9599e-01,  8.6603e-01,  8.3257e-01,
         7.9576e-01,  7.5575e-01,  7.1269e-01,  6.6677e-01,  6.1816e-01,
         5.6706e-01,  5.1368e-01,  4.5823e-01,  4.0093e-01,  3.4202e-01,
         2.8173e-01,  2.2031e-01,  1.5800e-01,  9.5056e-02,  3.1728e-02,
        -3.1728e-02, -9.5056e-02, -1.5800e-01, -2.2031e-01, -2.8173e-01,
        -3.4202e-01, -4.0093e-01, -4.5823e-01, -5.1368e-01, -5.6706e-01,
        -6.1816e-01, -6.6677e-01, -7.1269e-01, -7.5575e-01, -7.9576e-01,
        -8.3257e-01, -8.6603e-01, -8.9599e-01, -9.2235e-01, -9.4500e-01,
        -9.6384e-01, -9.7880e-01, -9.8982e-01, -9.9685e-01, -9.9987e-01,
        -9.9887e-01, -9.9384e-01, -9.8481e-01, -9.7181e-01, -9.5490e-01,
        -9.3415e-01, -9.0963e-01, -8.8145e-01, -8.4973e-01, -8.1458e-01,
        -7.7615e-01, -7.3459e-01, -6.9008e-01, -6.4279e-01, -5.9291e-01,
        -5.4064e-01, -4.8620e-01, -4.2979e-01, -3.7166e-01, -3.1203e-01,
        -2.5115e-01, -1.8925e-01, -1.2659e-01, -6.3424e-02, -2.4493e-16],
       dtype=torch.float64)
```

你也可以按照仓库里的做法, 先克隆然后直接在源码目录里 `uv run test-uv`.

不过值得一提的是, 我以前陷入了一个误区, 我先克隆了仓库, 然后又从 github 仓库安装, 最后又 `uv run` , 实际上那样做从 github 安装的步骤是无效的, 因为这个过程发生了两次 build , 第一次在`uv pip install git+` 时, 第二次发生在 `uv run` 时(它根据源码安装), 所以你总是在运行源码.

另外, `uv pip install git+` 虽然看上去和 `pip install git+` 非常地像, 但是, 它用的是 uv 自己的引擎, 速度是 pip 的好多倍.
