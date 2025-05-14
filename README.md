# 前言

关于这本"书":

我曾在 Linux.do 上面分享过自己探索 uv 的过程,中途收到了很多人的肯定: [探索 uv （完结！） | An extremely fast Python package and project manager, written in Rust.](https://linux.do/t/topic/457885)

但是其中实际上相当一部分现在在我看来有些不妥,而且我后来也写了很多关于 uv 的东西,即使现在对我都很有用,但似乎过于零散不是很好整理,所以这里就升起了写本"书"的想法.也是在写代码之余找点轻松的事情做.因为如果只是写代码,对我来说还是过于压抑的.毕竟我是个话痨.

这本书可能分为两部分,第一部分就是修改后的那个`探索 uv` 旨在让刚刚接触 uv 的人能够快速地入门. 避免跟我之前一样被语料库落后的大模型耽误整整一两个月甚至产生了恐惧心理.

> 关于这本书的呈现形式, 我考虑过两种, 第一种是 [github.io](https://github.com/nndl/nndl.github.io), 第二种是[vue press 还是 vite press 渲染静态网页(这对我来说似乎太复杂了)](https://yutto.nyakku.moe/), 第三种是 jupyter notebook 渲染静态网页, 第四种是 [github 直接的 .md 教程](https://github.com/RimoChan/Vtuber_Tutorial), 但目前的问题不在于形式,而在于内容.在写这个的过程中,我心里萦绕的总是村上说, 写长篇小说更像是一种体力劳动, 确实, 有时候即使想写的东西很多, 但是, 在还没有完全把想写的东西都写出来, 就已经精疲力尽没有灵感了.

# 第一部分: 探索 uv

> 为啥我找不到一个完整的关于 uv 的中文文档或者教程。
>
> 是 pip 的黑恶势力太强大了吗？不，应该只是人们对于接受未知的事物抱有恐惧心理。
>
> 记录我的探索过程和路线，完全按照我的思路来, 所以可能在你看来会超级乱。

## 什么是 uv?

你把一个立方体的表面沿着一条线剪开，把所有的面展开，就得到了 uv 映射。你可以利用 uv 贴图来给模型贴上纹理。

…

呸呸呸，这里我们要讲的 uv，是一个包管理器, 当然你也可以仅仅把它作为一个超级快的 pip 来用.

![](images/uv_pip_speed.svg)

这是 uv 和 pip 的安装速度对比。

我们将在这里探索到一些 uv 的基础用法，取代 pip 的几种方式。以及会在后面尝试编译我自己的第一个 python package, 并且补充一下如何上传到 pypi .

## 你可以在阅读本文档的同时阅读的:

- [uv 官方文档 - 任何时候你碰到问题,你都可以查阅它,除了纯英文对我来说有点困难它没有缺点,你也可以考虑用联网的大模型让帮你查找问题解法.](https://docs.astral.sh/uv/getting-started/installation/)

> 但是我个人更建议自己查找噢.

## uv.lock + pyproject.toml vs requirements.txt   

> 这个部分的所有内容几乎都会在我们的练习实践里体现, 如果你是动手党, 不如直接跳到练习实践部分.因为我本身也是一个动手党,所以我理解看大篇结论是多么无聊的事情.你完全可以跳过这一节, 也许你在实践所有部分后, 得出跟我类似的结论,不过那个时候你才掌握了它.

相信大部分人都是从 `conda + pip + reqirements.txt` 的时代过来的, 以及可能从来没有过`pyproject` 维护经验的, 如果你是从 `poetry` 等和 `uv` 用法相似的时代过来的,并且已经有了丰富的包管理经验和 `pyproject` 维护经验,那么你可以直接跳到本书的第二部分.

相比于 `requirements.txt` , `uv.lock` + `pyproject.toml` 意味着三点极大的优势:

- 跨平台的良好兼容性 (虽然这点本身考验维护者)
- 自由自定义依赖库 (你可以从 github 安装你自己写的依赖库版本,可以从本地实时编译最新的依赖库版本, pypi 这样的官方源只是一种选项,它具有极大的自由度)
- 简单的包编译和上传(运行即编译, 你不需要写和调试复杂的 setup.py , all you need is ,保证你的代码可以用 uv 运行,只要能运行,就能编译和上传,丝毫不需要为安装费脑子)
  > 还有更多这里暂时不提了, 比如说, 在多个 package 共存时,存在多种方式可以让快速的源码同步实时编译运行等等.

### 关于跨平台兼容性:

它主要是因为 `uv.lock`

```shell
uv.lock is a human-readable TOML file but is managed by uv and should not be edited manually. 
uv.lock 是一个人类可读的 TOML 文件，但由 uv 管理，不应手动编辑。
```

这是一段截取的 [uv.lock](https://github.com/MrXnneHang/XnneHangLab/blob/dev/uv.lock):

```toml
[[package]]
name = "funasr"
version = "1.2.6"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "editdistance" },
    { name = "hydra-core" },
    { name = "jaconv" },
    { name = "jamo" },
    { name = "jieba" },
    { name = "kaldiio" },
    { name = "librosa" },
    { name = "modelscope" },
    { name = "oss2" },
    { name = "pytorch-wpe" },
    { name = "pyyaml" },
    { name = "requests" },
    { name = "scipy" },
    { name = "sentencepiece" },
    { name = "soundfile" },
    { name = "tensorboardx" },
    { name = "torch-complex" },
    { name = "tqdm" },
    { name = "umap-learn" },
]
sdist = { url = "https://files.pythonhosted.org/packages/4d/76/7b550401bd91a50dd6988615f28e701d69aa96f91ff538e7b6555baf31ed/funasr-1.2.6.tar.gz", hash = "sha256:702995804b87c6c8d591702f660ad64a50c09127de5e1b28556faa9bd9db8588", size = 553274 }
wheels = [
    { url = "https://files.pythonhosted.org/packages/1c/8c/af5817acd4ad5c9b97aafb7c5214cdd84a6c71eaac7c733d18e8449c3f47/funasr-1.2.6-py3-none-any.whl", hash = "sha256:74f339fa92d252e17616758bc94666908d02356281690e1e242110ac6676800c", size = 701567 },
]
```

这是我截取的我的一段 requirements.txt...

```txt
selenium
webdriver_manager
bs4
pyaml
lxml
```

uv.lock 通过运行 `uv lock` 根据 pyproject.toml 生成.它准确到代码的哈希, 而 requirements.txt, 它根据我的心情生成, 在我心情好的时候,我给他把所有必要的包包括版本号都写出来,在我心情一般的时候,我帮他写不带版本号的,在我没心情的时候,他还是自己写吧.

所以仅仅从稳定性来说, `uv.lock` 就更可靠,而且它也更精确, 另外, 如果 `pyproject.toml` 没写好, 在运行 `uv lock` 的时候就会报错.

敲黑板: **很多时候 `uv lock`, 可以作为我们检查依赖的一个手段.虽然 uv lock 被集成在一些其他后续步骤里面, 但是手动的运行是非常推荐的.**

### 关于自由的依赖库自定义

```toml
[tool.uv.sources]
yutto = { git = "https://github.com/MrXnneHang/yutto.git", rev = "parse" }
# yutto = { path = "./packages/yutto/dist/yutto-2.0.3-py3-none-any.whl"}
# yutto = { workspace = True }
```

默认的依赖库安装源是 `pypi`, 而有时候, 你可能在同时维护两个库, 其中一个依赖于另外一个, 并且你希望能够快速地修改源代码和调试其中的一个依赖, 并且希望能够立刻运行和反映.而 uv 提供了几种依赖库的安装源:

- **github:** 比如你可以简单地 commit 提交到 github 上, 然后从 github 拉取自动安装 (但这个方式在快速调试的时候可能会因为读取的哈希没更新的而导致使用的不是最新的源代码.) **这个方式推荐在 release 到 pypi 前, 进行不同设备的迁移测试的时候使用.**
- **本地 whl**: 或者你可以用 `uv build` 来简单地生成一个 whl 包, 然后像从 pypi 上安装那样使用, 这点缺点是有时候编译显得耗时.但我习惯这么用,因为它在独立性和源码同步上都有相当不错的表现.通常我结合 `justfile` 食用. 比如我希望实时使用最新的 `./packages/yutto` 的代码,我会这么做:

```justfile
dev:
    # 删除所有构建产物和缓存 / 二次操作防止缓存问题恢复代码
    rm -rf packages/*/dist
    rm -rf packages/*/__pycache__
    rm -rf packages/*/*.egg-info
    uv build packages/yutto
    uv lock --no-cache
    uv run streamlit run src/uiya/yutto_uiya.py # 在这里运行你的主程序.
```

然后这么使用:

```shell
just dev
```

- **workspace:** 这点是 uv 的特点,但是内容比较多,会放在后面的专题来讲. 它的特点在源码同步完美, 速度完美, 不过, 在维护代码的时候, 有时候不是很好处理.

### 简单的包编译和 pypi 上传

这点可以我最近重构的一个仓库那里看出.

原仓库地址: [https://github.com/raczben/wexpect](https://github.com/raczben/wexpect)

作者年久失修, 并且留下一些挺大的坑没有在最新的 release 中修复,具体参见 : [https://github.com/XnneHangLab/yutto-uiya/issues/4](https://github.com/XnneHangLab/yutto-uiya/issues/4).

pexpect 在 linux 等 unix 系统中表现良好, 但是无法在 windows 中使用, 而我又非常需要它的某些特性,于是乎我找上了平替 `wexpect` , 但是它和我预想有些不一样, 比如它无法在 uv 的 `venv` 下工作, 在 conda 中正常工作. 捕获 `\r` 相关的行的时候无法正常检测到换行. 并且也无法正常解析出终端字体颜色属性为 ASCII 码.

我在 fork 它后, 用 uv 重构了一下, 即使我一点也看不懂它原本的编译方式, 但是并不妨碍我简单地写了一个 `pyproject.toml` , 然后补齐它的依赖后, 让可以正常运行后, 就可以编译了. 我也很高兴最近有人注意到我这个仓库并且尝试在我的修改上继续修改.

这是我的 [pyproject.toml](https://github.com/XnneHangLab/wexpect-uv/blob/dev/pyproject.toml):

```toml
[project]
name = "wexpect-uv"
version = "0.0.4"
description = "Windows alternative of pexpect rebuild by uv"
readme = "README.md"
requires-python = ">=3.9,<3.14"
license = {text = "MIT"}
authors = [
    {name = "Noah Spurrier, Richard Holden, Marco Molteni, Kimberley Burchett, Robert Stone, Hartmut Goebel, Chad Schroeder, Erick Tryzelaar, Dave Kirby, Ids vander Molen, George Todd, Noel Taylor, Nicolas D. Cesar, Alexander Gattin, Geoffrey Marshall, Francisco Lourenco, Glen Mabey, Karthik Gurusamy, Fernando Perez, Corey Minyard, Jon Cohen, Guillaume Chazarain, Andrew Ryan, Nick Craig-Wood, Andrew Stone, Jorgen Grahn, Benedek Racz", email = "betontalpfa@gmail.com"},
    {name = "XnneHang", email = "XnneHang@gmail.com"}
]
dependencies = [
    "pywin32>=220",
    "psutil>=5.0.0",
    "setuptools>=76.0.0",
    "wcwidth>=0.2.13",
]
keywords = [
    "scripting", "automation", "expect", "pexpect", "wexpect"
]
classifiers = [
    "Development Status :: 4 - Beta",
    "Environment :: Console",
    "Intended Audience :: Developers",
    "Intended Audience :: Information Technology",
    "Operating System :: Microsoft :: Windows",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
]

[project.urls]
"Source Code" = "https://github.com/XnneHangLab/wexpect-uv"

[dependency-groups]
test = [
    # "coverage",
    "pyinstaller>=5.6.2",

]

dev = [
   "pyright>=1.1.398",
   "ruff>=0.11.4",
   "typos>=1.31.1",
   "pytest>=8.3.5",
   "pytest-rerunfailures>=15.0",
  "syrupy>=4.9.0",
   "pytest-codspeed>=3.2.0",
]

[project.scripts]
wexpect = "wexpect.__main__:main"

[tool.flake8]
max-line-length = 100
ignore = ["E402"]

# If your package is a simple module (py_modules=['wexpect'])
[tool.hatch.build.targets.wheel]
packages = ["src/wexpect"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

我想说的是,非常不可思议的, 我不需要写一个复杂的 `setup.py`, 也不需要像 `C++` 那样写一个复杂的 `CMakeLists.txt`, 只需要写一个简单的 `pyproject.toml` , 写清楚依赖, 保证我的程序可以正常使用 `uv run` 运行,那么我就可以正常打包和上传.

所以,包编译和 pypi 上传反而是最简单的.因为它几乎没有任何额外工作.

我们也会在探索的最后, 练习一下如何上传到 pypi.

## 不妨实践

### 实践之前

安装 uv 以及最终建议( 使用前你总是需要先安装 )

请参考[官方文档](https://docs.astral.sh/uv/getting-started/installation/)进行安装.

我最终建议是全局安装 uv + 使用 pyproject.toml + uv.lock 的方式来管理项目（python lib）。

具体参考:

https://github.com/XnneHangLab/wexpect-uv

https://github.com/yutto-dev/yutto

https://github.com/XnneHangLab/XnneHangLab

或者图方便只是用来加速安装也可以简单用 uv+conda 来替代 pip+conda。用法上，就是把所有的:

pip install → uv pip install

### 如何运行一个 uv 项目

更多时候我们并不是从零开发一个项目而是使用或者试图向别人的项目 PR .

那么这里就用上面三个例子为例.

#### [wexpect-uv](https://github.com/XnneHangLab/wexpect-uv.git)

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

#### [yutto](https://github.com/yutto-dev/yutto.git)

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

#### [XnneHangLab](https://github.com/XnneHangLab/XnneHangLab.git)

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
