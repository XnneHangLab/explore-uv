<h1 align="center">使用 pypi 镜像源安装 python package</h1>  

- [官方文档里的词条](https://docs.astral.sh/uv/configuration/indexes/#pinning-a-package-to-an-index)<br>
- [参考仓库:test-uv-index](https://github.com/MrXnneHang/test-uv-index)

有时候我们可能没有办法连接外网, 或者我们的用户无法连接外网, 那么在这种网络限制下, 我们必须要使用镜像源来安装 python package.<br>

之前我们提到[修改 `tool.uv.index` 指定下载 cpu 或者 cuda 的 torch 并且使用镜像](../chapters/pytorch_index_mirror.md)

当时我们发现, torch 指定的是一个独立于 pypi 的`index-url`，而实际上，我们在 pip 换源的时候，换的也是`index-url`。<br>

诸如：<br>

```txt
# 官方源
https://pypi.org/simple
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

## 项目级配置

可以很容易得出，我们可以这样配置 `pyproject.toml`:<br>

```toml
dependencies = [
     "numpy==1.26.4",
     "matplotlib==3.10.0"
]
[[tool.uv.index]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
default = true
```

不同的是, 我们这里的`default = true`, 而上次用的是 `explicit = true`.<br>

这里的区别是, `explicit = true` 需要额外在 `[tool.uv.sources]` 里面进行指定才会生效, 否则它不会被用到, 而如果设置为 default, 那么意味着它将取代原本的默认源(pypi) 来下载包.<br>

至于一个项目能不能同时写两个 default , 这个邪恶的想法 uv 也很早料到, 它不会抛出错误, 而是根据优先级, 似乎上面的优先, 会覆盖下面的, 但我也不记得了, 不建议尝试, 如果担心一个源不能用, 也可以加上一些其他源作为被选, 这么加:

```toml
[[tool.uv.index]]
name = "aliyun"
url = "https://mirrors.aliyun.com/pypi/simple/"
```

不加`default = true` 也不加 `explicit = true`.代表作为备选.<br>

这里是一个简单的测试仓库: [MrXnneHang/test-uv-index](https://github.com/MrXnneHang/test-uv-index)

你可以简单地克隆和进行尝试.

## 用户级配置

项目级配置应该是最常用且复杂和繁琐的, 我个人也只建议进行项目级配置, 因为很多时候, 我们的项目是给别人用, 而别人的 uv 是默认配置, 如果我们的用户配置造成了和别人差异化, 那么在项目移植和发布的时候就会造成一些不必要的麻烦.<br>

用户级的配置指的是配置 `~/.config/uv/uv.toml`.<br>

[参考这里](https://linux.do/t/topic/471143/9?u=xnnehang)

文件似乎得自己创建.

我一般不用也不建议用, 所以这里不做拓展.

## 系统级配置

`export` 等环境变量的配置. 或者 `uv` 命令后面加的 `--index-url` 这样的参数.

比较耐人寻味的是如果同时配置了三个层级, 优先级哪个最高. 因为而这点存疑, 所以我一般只配置 `pyproject.toml`.我也只建议配置 `pyproject.toml`. 避免在给别人用的时候因为 uv 本身的配置不同造成无法运行.
