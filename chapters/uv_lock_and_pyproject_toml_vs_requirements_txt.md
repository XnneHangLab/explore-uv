<h1 align="center">uv.lock + pyproject.toml vs requirements.txt</h1>  

<p align="center">
  <a href="#跨平台兼容性"><strong>跨平台兼容性</strong></a> ·
  <a href="#自由的依赖库自定义"><strong>自由的依赖库自定义</strong></a> ·
  <a href="#简单的包编译和上传"><strong>简单的包编译和上传</strong></a>
</p>

> 这个部分的所有内容几乎都会在我们的练习实践里体现, 如果你是动手党, 不如直接跳到练习实践部分.因为我本身也是一个动手党,所以我理解看大篇结论是多么无聊的事情.你完全可以跳过这一节, 也许你在实践所有部分后, 得出跟我类似的结论,不过那个时候你才掌握了它.

相信大部分人都是从 `conda + pip + reqirements.txt` 的时代过来的, 以及可能从来没有过`pyproject` 维护经验的, 如果你是从 `poetry` 等和 `uv` 用法相似的时代过来的,并且已经有了丰富的包管理经验和 `pyproject` 维护经验,那么你可以直接跳到本书的第二部分.

相比于 `requirements.txt` , `uv.lock` + `pyproject.toml` 意味着三点极大的优势:

- 跨平台的良好兼容性 (虽然这点本身考验维护者)
- 自由自定义依赖库 (你可以从 github 安装你自己写的依赖库版本,可以从本地实时编译最新的依赖库版本, pypi 这样的官方源只是一种选项,它具有极大的自由度)
- 简单的包编译和上传(运行即编译, 你不需要写和调试复杂的 setup.py , all you need is ,保证你的代码可以用 uv 运行,只要能运行,就能编译和上传,丝毫不需要为安装费脑子)
  > 还有更多这里暂时不提了, 比如说, 在多个 package 共存时,存在多种方式可以让快速的源码同步实时编译运行等等.
  > 再比如全面的镜像源支持, 从 python 下载, 到包安装, 到一些如 pytorch 的库, 你都可以为它自定义下载源.非常地照顾国内用户的网络环境.

## 跨平台兼容性

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

这个跨平台兼容性也正是因为 `uv.lock`, 它会根据你当前的操作系统, 选择合适的依赖库版本, 当然, 有时候会找不到, 比如 `wexpect` 依赖了 win32api, 那么它就只能在 windows 下使用, 当然这个也是可控的, 比如在 windows 下用 `wexpect-uv`, 在 unix-like 下用 `pexpect` 进行互补.

```shell
"pexpect==4.9.0;sys_platform!='win32'",
"wexpect-uv==0.0.4;sys_platform=='win32'",
```

这个跨平台兼容性的意义在于, 只要开发者有计划, 它可以让用户在任何系统下运行它的程序, 使用相同的命令, 无需考虑依赖的安装和差异化: `uv lock`->`uv sync`->`uv run`.

当然,这个考验开发者本身的水平. 不过这个时候, 你就不必为了克隆一个的古早的仓库, 然后反反复复地调整依赖还是运行不起来, 而感到无奈了.有事,直接找维护者.

## 自由的依赖库自定义

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

另外, 因为 uv 包编译和 pypi 上传变得简单, 你也可以为自己修改后的包单独 release 到 pypi .

## 简单的包编译和上传

你可以很容易地用 uv 来管理包, 并且上传 pypi.

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
