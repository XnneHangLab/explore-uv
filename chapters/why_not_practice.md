# 不妨实践

<p align="center">
  <a href="#实践之前(安装 uv 以及最终建议)"><strong>实践之前(安装 uv 以及最终建议)</strong></a> ·
  <a href="#如何运行一个 uv 项目"><strong>如何运行一个 uv 项目</strong></a> ·
</p>

## 实践之前(安装 uv 以及最终建议)

安装 uv 以及最终建议( 使用前你总是需要先安装 )

请参考[官方文档](https://docs.astral.sh/uv/getting-started/installation/)进行安装.

我最终建议是全局安装 uv + 使用 pyproject.toml + uv.lock 的方式来管理项目（python lib）。

具体参考:

https://github.com/XnneHangLab/wexpect-uv

https://github.com/yutto-dev/yutto

https://github.com/XnneHangLab/XnneHangLab

或者图方便只是用来加速安装也可以简单用 uv+conda 来替代 pip+conda。用法上，就是把所有的:

pip install → uv pip install

## 如何运行一个 uv 项目

更多时候我们并不是从零开发一个项目而是使用或者试图向别人的项目 PR .

那么这里就用上面三个例子为例.

### [wexpect-uv](https://github.com/XnneHangLab/wexpect-uv.git)

先克隆下来:

```shell
git clone https://github.com/XnneHangLab/wexpect-uv.git
cd wexpect-uv
```

首先, **依赖不需要手动安装**, 我们并不需要找到类似于 `requirements.txt` 这样的类似依赖, `uv.lock` 也并不需要人手动安装, **所有依赖都在运行时自动检查和安装**. 在我一次拿到 `yutto` 的时候, 大模型教我用 `uv install` , 应该是根据经验和 `npm` 的 `package.lock` 混淆了,我被误导得不清.

我们需要关注的仅仅只有 `pyproject.toml`.

第一步我们只需要看两个部分: `dependencies` 和 `project.scripts`

```toml
dependencies = [
    "pywin32>=220",
    "psutil>=5.0.0",
    "setuptools>=76.0.0",
    "wcwidth>=0.2.13",
]

[project.scripts]
wexpect = "wexpect.__main__:main"
```

简单地看一下就可以知道, 这个包只能在 windows 下使用, 因为它依赖于 `pywin32` .

并且如果它具有 `project.scripts` , 代表它可以作为一个类似于 `ffmpeg` 这样的命令行工具使用.

我们直接运行 `uv run wexpect -h` 看看:

```shell
➜  wexpect-uv git:(dev) uv run wexpect -h
Using CPython 3.13.0
Creating virtual environment at: .venv
error: Distribution `pywin32==310 @ registry+https://pypi.org/simple` can't be installed because it doesn't have a source distribution or wheel for the current platform

hint: You're on Linux (`manylinux_2_38_x86_64`), but `pywin32` (v310) only has wheels for the following platforms: `win32`, `win_amd64`, `win_arm64`
```

> 不好意思, 目前环境是 linux , 等有时间了再补一下.

当然有的包并不是作为可执行程序使用, 而是作为一个库 `import` 后使用.

那么可以先这么进行测试:

```shell
uv run python

>>> import wexpect
>>> wexpect.__version__
'0.0.4'
```

或者:

```shell
uv run test.py # 你的测试文件,里面包含一些对 wexpect 的一些用例
```

到这里, 我们就可以简单地使用这个包了, 或者修改一些源代码后进行测试. `uv run script` 每次在运行前都会自动检查和安装依赖, 并且, 总是在最新的源代码下运行. 如果你需要增添依赖,那么可以直接修改 `pyproject.toml` 然后再次运行 `uv run script` .

`uv run` 很多时候似乎是万能的.

因为它集成度很高. 实际上这个过程大概分为 `uv lock`(检查和更新 lock 文件) -> `uv sync`(根据 lock 文件安装依赖) -> `uv run`(运行脚本) . 如果单独运行 `uv sync`, 那么它也会集成 `uv.lock` 自动生成或者更新, `uv.lock`.

对于复杂的环境, 通常推荐拆分运行:

```shell
uv lock
uv sync
uv run script
```

你也可以简单地用 `justfile` 来运行:

```justfile
dev:
    uv lock
    uv sync
    uv run script
```

然后使用 `just dev` 运行.

之所以拆分 `lock` , `sync` , `run` 是因为它可以让我们清楚地知道错误发生在哪个环节, 是依赖项检查还是依赖安装还是运行时报错.

但对于简单的项目, 直接 `uv run script` 就可以了.

### [yutto](https://github.com/yutto-dev/yutto.git)

对于 `yutto` , 大部分都和 `wexpect-uv` 一致.

唯一比较特殊的是它使用到了 rust, 于是在构建时需要安装 rust 工具链. 这些可以参阅作者可能提供的 [CONTRIBUTING.md](https://github.com/yutto-dev/yutto/blob/main/CONTRIBUTING.md).

于是我们的运行步骤变成了:

```shell
git clone https://github.com/yutto-dev/yutto.git
cd yutto
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh # 安装 rust 工具链
uv run yutto -h
```

我们并不需要为 rust 的引入写更多的代码或者导致运行不一致, 只需要保证 `pyproject.toml` 里面编写正确即可.

### [XnneHangLab](https://github.com/XnneHangLab/XnneHangLab.git)

这是一个综合性的仓库. 它本身使用 uv 管理, 而它有几个特殊的依赖库, 也都在我的管理或者参与下.并且也都是使用 uv 管理. 我对 uv 的理解也都是在这个仓库中逐渐形成的.

对于所有的依赖库, `wexpect-uv` 从 pypi 安装, `yutto` 从 github 安装, `yutto-uiya` 作为 workspace 成员.

而在快速开发时, `wexpect-uv` 和 `yutto` 均实时使用 `uv build` 从最新的源代码构建 `whl` 包然后安装. `yutto-uiya` 作为 workspace 不需要编译, 直接使用最新的源代码.

即使它看上去很唬人, 而且这个仓库中也引入了之前没有接触过的 `[uv.index]` 以及 `[uv.sources]`, 以及为不同的操作系统提供不同的依赖项, 比如在 unix-like 的 系统中使用 `pexpect` 而在 windows 中使用 `wexpect`.

但即使加了这么多东西, 我们运行的逻辑依然是一样的:

这是我稳定版本运行的命令.

```shell
uv lock
uv sync
uv run get_root
uv run streamlit run src/lab/ui.py
```

这是我开发版本运行的命令:

```shell
# 删除所有构建产物和缓存 / 二次操作防止缓存问题恢复代码
rm -rf packages/*/dist
rm -rf packages/*/__pycache__
rm -rf packages/*/*.egg-info
uv build packages/yutto
uv build packages/wexpect-uv
uv lock --no-cache --upgrade
uv run get_root
uv run streamlit run src/lab/ui.py --server.port 8000
```

可以说本质还只是 `uv run script` , 之所以要 `uv run streamlit run` , 是因为 Streamlit 的 WebUI 进程要在 `streamlit run` 的子进程中运行, 而不是在 `uv run python` 的子进程中运行.

这里也可以再次补充一下, `uv run` 的运行对象, 可以是 `.py` , 可以是 `pyproject.script` , 也可以是一些存在于`.venv/bin/` 下的可执行程序如 `pip`, `python`, `yutto`, `streamlit`.

如果以这个角度去区分, 就很好理解 `uv run pip instal` 和 `uv pip install` 的区别了. 前者只是使用当前环境内的 pip 来安装依赖, 而后者则是使用 uv 自己的引擎来安装依赖, 所以如果要加速安装依赖, 你应该使用 `uv pip install` 而不是 `uv run pip install` . 之所以取名为 `pip` 也许只是一种妥协, 认为所有人更容易接受 `pip` , 而 `uv install` 本身是用来安装 `python`解释器到全局环境内的.这点和 `npm` 完全不同. 在与 uv 一般不需要手动运行 `npm install` 这样的命令进行依赖安装, 即使要, 对应的命令也应该是 `uv sync` 而不是 `uv install` .

会运行只是第一步, 有时候你可能需要微调一些东西来让自己运行起来, 所以还是有必要看懂 `pyproject.toml` 的里的一些必要条目的.
