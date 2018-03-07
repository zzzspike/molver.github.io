---
title: "composer 私有库搭建及使用方法"
date:   2018-3-15 14:06:00 +0800
categories: 编程人生
tags: php
---

为便于公司内部代码使用 `composer` 进行包管理，搭建了一个 `composer` 私有库。有三种方案可以搭建：
- `packagist`
- `satis`
- `artifact`

综合考量，使用轻量级的方案 `satis` 更符合需求。下面介绍下如何制作 `composer` 包，如何用 `satis` 搭建 `composer` 私有库，以及如何使用搭建的私有库。
<!--more-->

## composer 包基础知识
### 每个项目都是一个包
只要你有一个 composer.json 文件在目录中，那么整个目录就是一个包。当你添加一个 require 到项目中，你就是在创建一个依赖于其它库的包。你的项目和库之间唯一的区别是，你的项目是一个没有名字的包。

为了使它成为一个可安装的包，你需要给它一个名称。你可以通过 composer.json 中的 name 来定义：
~~~ json
{
    "name": "company/example",
    "require": {
        "yiisoft/yii2": ">=2.0.6"
    }
}
~~~
### 指定版本
`satis` 能够从 VCS (git, svn, hg) 的信息中推断出包的版本，因此不必手动指明版本号，并且也不建议这样做。请查看 **标签** 和 **分支** 来了解版本号是如何被提取的。

#### 标签
对于每一个看起来像版本号的标签，都会相应的创建一个包的版本。它应该符合 'X.Y.Z' 或者 'vX.Y.Z' 的形式，`-patch`、`-alpha`、`-beta` 或 `-RC` 这些后缀是可选的。在后缀之后也可以再跟上一个数字。

下面是有效的标签名称的几个例子：
 `1.0.0`
`v1.0.0`
`1.10.5-RC1`
`v4.4.4beta2`
`v2.0.0-alpha`
`v2.0.4-p1`
> 注意： 即使你的标签带有前缀 `v`， 由于在需要 `require` 一个版本的约束时是不允许这种前缀的， 因此 `v` 将被省略（例如标签 `V1.0.0` 将创建 `1.0.0` 版本）。

#### 分支
对于每一个分支，都会相应的创建一个包的开发版本。如果分支名看起来像一个版本号，那么将创建一个如同 `{分支名}-dev` 的包版本号。例如一个分支 `2.0` 将产生一个 `2.0.x-dev` 包版本（加入了 `.x` 是出于技术的原因，以确保它被识别为一个分支，而 2.0.x 的分支名称也是允许的，它同样会被转换为 `2.0.x-dev`）。如果分支名看起来不像一个版本号，它将会创建 `dev-{分支名}` 形式的版本号。例如 `master` 将产生一个 `dev-master` 的版本号。

下面是版本分支名称的一些示例：
`1.x`
`1.0` (等同于 `1.0.x`)
`1.1.x`

## 使用 satis 搭建私有库
- 下载 satis

```composer create-project composer/satis --stability=dev```

- 配置安装

在 satis 目录下新建 satis.json 文件，输入配置如下：
```json
{
    "name": "你的仓库名称",
    "homepage": "http://packages.example.org", //仓库url地址
    "description": "资源包详情描述",
    "repositories": [
        { "type": "vcs", "url": "http://github.com/mycompany/privaterepo" },//需要抓取资源包的vcs地址1
        { "type": "vcs", "url": "http://github.com/mycompany/privaterepo2" }//需要抓取资源包的vcs地址2
    ],
    "require-all": true
}

```
配置完成后，运行 `php bin/satis build satis.json web/` （会提示输入私有vcs账号密码，输入并将其保存到本地），在web目录下会生成静态私有库。将 web 目录配置到 http 服务器上即可。

另外还需写个脚本，在 vcs 库推送新代码、新tag或新分支时，运行`php bin/satis build --repository-url vcs地址 satis.json web/'`更新相应 `vcs` 地址的静态私有库。

## 如何使用私有库
在项目 `composer.json` 文件中加入
~~~ json
{
  "repositories": [{
    "type": "composer",
    "url": "https://packages.example.org"//私有库地址
  }]
}
~~~
运行 `composer require 包名:版本号` 即可引入私有库中的包（会提示输入 cvs 的账号密码，建议保存到本地，后续就不再提示）

> 访问私有库地址，可查看库里包含资源包列表

## 私有库的维护
- 每个资源包对应一个 vcs 地址，以便于管理版本（通过 tag 和 branch 管理）。
- 添加资源包：将资源包对应 vcs 地址添加进 satis.json文件，并运行 `php bin/satis build satis.json web/`更新私有库。
- 更新资源包：已有脚本运行自动更新，无需处理。

