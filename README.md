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

## [uv.lock + pyproject.toml vs requirements.txt](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/uv_lock_and_pyproject_toml_vs_requirements_txt.md)

- [跨平台兼容性](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/uv_lock_and_pyproject_toml_vs_requirements_txt.md#跨平台兼容性)
- [自由的依赖库自定义](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/uv_lock_and_pyproject_toml_vs_requirements_txt.md#自由的依赖库自定义)
- [简单的包编译和上传](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/uv_lock_and_pyproject_toml_vs_requirements_txt.md#简单的包编译和上传)

## [不妨实践](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/why_not_practice.md)

- [实践之前(安装方式和最终建议)](<https://github.com/XnneHangLab/explore-uv/tree/master/chapters/why_not_practice.md#实践之前(安装方式和最终建议)>)
- [如何运行别人的项目](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/why_not_practice.md#如何运行别人的项目)
  - [wexpect-uv](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/why_not_practice.md#wexpect-uv)
  - [yutto](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/why_not_practice.md#yutto)
  - [yutto-uiya](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/why_not_practice.md#yutto-uiya)
  - [XnneHangLab](https://github.com/XnneHangLab/explore-uv/tree/master/chapters/why_not_practice.md#XnneHangLab)

# 第二部分: 更深入地解决问题

- [使用镜像安装 python | python-install-mirror](chapters/python_install_mirror.md)
