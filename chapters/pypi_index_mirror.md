## Refrence:

- [https://docs.astral.sh/uv/configuration/indexes/#pinning-a-package-to-an-index](https://docs.astral.sh/uv/configuration/indexes/#pinning-a-package-to-an-index)<br>
- [参考仓库:test-uv-index](https://github.com/MrXnneHang/test-uv-index)

原因是我发现我的 uv 似乎吃不到代理.

我用 pip 直接下载的速度是 6MB/s,但是，我用 uv 下载一个 20MB 的 Numpy 下载时间要 7 分钟。<br>

当然,

当时我发现，我们 torch 指定的是一个`index-url`，而实际上，我们在 pip 换源的时候，换的也是`index-url`。<br>

诸如：<br>

```txt
#清华源
https://pypi.tuna.tsinghua.edu.cn/simple
# 阿里源
https://mirrors.aliyun.com/pypi/simple/
# 腾讯源
http://mirrors.cloud.tencent.com/pypi/simple
# 豆瓣源
http://pypi.douban.com/simple/
```

这些实际上也都是 index-url。于是乎我们参考上一次配置 torch 的 index-url：

```toml
dependencies = [
    "funasr==1.2.4",
    "pyaml==25.1.0",
    "torch==2.1.0",
    "torchaudio==2.1.0",
]

[[tool.uv.index]]
name = "pytorch-cpu"
url = "https://download.pytorch.org/whl/cpu"
explicit = true

[tool.uv.sources]
torch = [
  { index = "pytorch-cpu" },
]
torchaudio = [
  { index = "pytorch-cpu" },
]
```

可以很容易得出，我们可以这样配置镜像源:<br>

```toml
dependencies = [
     "numpy==1.26.4",
     "matplotlib==3.10.0"
]
[[tool.uv.index]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
explicit = true

[[tool.uv.index]]
name = "aliyun"
url = "https://mirrors.aliyun.com/pypi/simple/"
explicit = true

[tool.uv.sources]
numpy = [
      { index = "tsinghua"},
]

matplotlib = [
      { index = "aliyun"}
]
```

意思是我们用阿里源下载 matplotlib ,用清华源下载 numpy。<br>

然后我们再写一个简单的脚本测试一下。<br>

[https://github.com/MrXnneHang/test-uv-index](https://github.com/MrXnneHang/test-uv-index)

但是这么做，我发现，`uv.lock`中，我们指定的包确实都转向了`tsinghua`和`aliyun`，但是所有的依赖包并不会被解析为使用镜像源，依然采用`pypi.org`.

```txt
[[package]]
name = "matplotlib"
version = "3.10.0"
source = { registry = "https://mirrors.aliyun.com/pypi/simple/" }
dependencies = [
    { name = "contourpy" },
    { name = "cycler" },
    { name = "fonttools" },
    { name = "kiwisolver" },
    { name = "numpy" },
    { name = "packaging" },
    { name = "pillow" },
    { name = "pyparsing" },
    { name = "python-dateutil" },
]
sdist = { url = "https://mirrors.aliyun.com/pypi/packages/68/dd/fa2e1a45fce2d09f4aea3cee169760e672c8262325aa5796c49d543dc7e6/matplotlib-3.10.0.tar.gz", hash = "sha256:b886d02a581b96704c9d1ffe55709e49b4d2d52709ccebc4be42db856e511278" }
wheels = [
    { url = "https://mirrors.aliyun.com/pypi/packages/09/ec/3cdff7b5239adaaacefcc4f77c316dfbbdf853c4ed2beec467e0fec31b9f/matplotlib-3.10.0-cp310-cp310-macosx_10_12_x86_64.whl", hash = "sha256:2c5829a5a1dd5a71f0e31e6e8bb449bc0ee9dbfb05ad28fc0c6b55101b3a4be6" },
    ...

[[package]]
name = "numpy"
version = "1.26.4"
source = { registry = "https://pypi.tuna.tsinghua.edu.cn/simple" }
sdist = { url = "https://pypi.tuna.tsinghua.edu.cn/packages/65/6e/09db70a523a96d25e115e71cc56a6f9031e7b8cd166c1ac8438307c14058/numpy-1.26.4.tar.gz", hash = "sha256:2a02aba9ed12e4ac4eb3ea9421c420301a0c6460d9830d74a9df87efa4912010", size = 15786129 }
wheels = [
    { url = "https://pypi.tuna.tsinghua.edu.cn/packages/a7/94/ace0fdea5241a27d13543ee117cbc65868e82213fb31a8eb7fe9ff23f313/numpy-1.26.4-cp310-cp310-macosx_10_9_x86_64.whl", hash = "sha256:9ff0f4f29c51e2803569d7a51c2304de5554655a60c5d776e35b4a41413830d0", size = 20631468 },
    ...

[[package]]
name = "packaging"
version = "24.2"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/packages/d0/63/68dbb6eb2de9cb10ee4c9c14a0148804425e13c4fb20d61cce69f53106da/packaging-24.2.tar.gz", hash = "sha256:c228a6dc5e932d346bc5739379109d49e8853dd8223571c7c5b55260edc0b97f", size = 163950 }
wheels = [
    { url = "https://files.pythonhosted.org/packages/88/ef/eb23f262cca3c0c4eb7ab1933c3b1f03d021f2c48f54763065b6f0e321be/packaging-24.2-py3-none-any.whl", hash = "sha256:09abb1bccd265c01f4a3aa3f7a7db064b36514d2cba19a2f694fe6150451a759", size = 65451 },
]
```

这是因为什么？我发现，这是因为我英语不好。<br>

`explicit = true`是排除的意思。<br>

```txt
An index can be marked as explicit = true to prevent packages from being installed from that index unless explicitly pinned to it. For example, to ensure that torch is installed from the pytorch index, but all other packages are installed from PyPI, add the following to your pyproject.toml:
可以将一个索引标记为 explicit = true ，以防止软件包从该索引安装，除非明确地钉在该索引上。例如，要确保 torch 从 pytorch 索引安装，而所有其他软件包都从 PyPI 安装，可在 pyproject.toml ：
```

所以，在我们只希望偶尔指定一下，那么我们用`explicit = true`参数。那么在正常的时候，它还是用我们的`pypi`。<br>

以及，它还提供了一个`defualt`参数。<br>

```txt
By default, uv includes the Python Package Index (PyPI) as the "default" index, i.e., the index used when a package is not found on any other index. To exclude PyPI from the list of indexes, set default = true on another index entry (or use the --default-index command-line option):
默认情况下，uv 将 Python 软件包索引 (PyPI) 列为 "默认 "索引，即在其他索引中找不到软件包时使用的索引。要从索引列表中排除 PyPI，可在其他索引条目中设置 default = true （或使用 --default-index 命令行选项）：

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
default = true
```

defualt 参数会把`pypi`源直接取代掉并且排除它。不再使用`pypi`源，即使在你设置的镜像源中找不到包，也不会搜索`pypi`。<br>

如果只设置了`defualt`的话实际上这存在一些问题。因为有时候一些包没有被及时维护更新到，或者镜像站偶尔抽风了就会导致你无法正常工作。<br>

`[tool.uv.index]`**还有不带参数的选项。**即这么写：<br>

```toml
[[tool.uv.index]]
# Optional name for the index.
name = "pytorch"
# Required URL for the index.
url = "https://download.pytorch.org/whl/cpu"


Indexes are prioritized in the order in which they’re defined, such that the first index listed in the configuration file is the first index consulted when resolving dependencies, with indexes provided via the command line taking precedence over those in the configuration file.
索引按其定义顺序排列优先级，因此在解析依赖关系时，配置文件中列出的第一个索引是第一个被查询的索引，通过命令行提供的索引优先于配置文件中的索引。

By default, uv includes the Python Package Index (PyPI) as the "default" index, i.e., the index used when a package is not found on any other index. To exclude PyPI from the list of indexes, set default = true on another index entry (or use the --default-index command-line option):
默认情况下，uv 将 Python 软件包索引 (PyPI) 列为 "默认 "索引，即在其他索引中找不到软件包时使用的索引。要从索引列表中排除 PyPI，可在其他索引条目中设置 default = true （或使用 --default-index 命令行选项）：
```

就是不带`default`和`explicit`参数.这样就像什么，就像 `conda channels`:<br>

```shell
channel URLs : https://repo.anaconda.com/pkgs/main/linux-64
              https://repo.anaconda.com/pkgs/main/noarch
              https://repo.anaconda.com/pkgs/r/linux-64
              https://repo.anaconda.com/pkgs/r/noarch
```

它会根据你定义的顺序作为它的优先级顺序，最上面的最优先，然后都找不到的话，就会使用`default`，默认是`pypi`，可以自己设定。你也可以用`explicit`来为特定的包指定 index, 但需要指定 source，像这样：<br>

```
[tool.uv.sources]
torch = [
  { index = "pytorch-cpu" },
]
torchaudio = [
  { index = "pytorch-cpu" },
]
```

相当完美和简洁的设计。<br>

于是乎最终我们定下来的配置是这样的：<br>

```shell
[[tool.uv.index]]
# Optional name for the index. 这个可以不写
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true

[[tool.uv.index]]
name = "aliyun"
url = "https://mirrors.aliyun.com/pypi/simple/"
```

我们用清华源替代 pypi 并且排除 pypi,然后给了 aliyun 作为备胎。当我们的清华源（它定义在 aliyun 上面，所以优先级更高）找不到时，就会尝试在阿里源进行搜索。<br>

可以看到[https://github.com/MrXnneHang/test-uv-index/blob/master/uv.lock](https://github.com/MrXnneHang/test-uv-index/blob/master/uv.lock)<br>

这次 uv.lock 中，清华源是我们的主角。=-=<br>

## PS:

上面提到的是利用 pyproject.toml 来替换。

uv index-url 的可以全局替换!

参见: https://linux.do/t/topic/471143/9?u=xnnehang

如果是个人使用的话，可以直接全局替换 uv 的镜像源，不需要单独配置 `pyproject.toml` 了。

也可以告诉用户，对方可以通过其他源来安装。如果提供的官方源配置的 `pyproject.toml` 无法让对方成功安装依赖。

佬友讲得超级好 =-= 。
