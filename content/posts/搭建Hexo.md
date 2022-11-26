---
title: "搭建Hexo"
date: 2022-11-25T15:23:28+08:00
description: "Hexo的搭建与安装过程"
tags: ['hexo', '静态网站']
categories: '软件安装与配置'
---

## 前言

在这次hexo的第一篇就记录一下hexo的搭建过程，不得不说，npm就算换源下载也好慢。

官网[Hexo](https://hexo.io/zh-cn/)

基于nodejs和npm，所以要确保系统中安装了这些。

## npm安装配置

### npm安装

[Node.js (nodejs.org)](https://nodejs.org/en/)

```shell
$ node -v
v12.22.9
$ npm -v
8.5.1
```

### npm换源

```shell
# 淘宝源
npm config set registry https://registry.npmmirror.com/
# 腾讯源
npm config set registry http://mirrors.cloud.tencent.com/npm/
```

查看当前镜像

```shell
npm config get registry
```

官方镜像

```shell
npm config set registry https://registry.npmjs.org
```

或者使用 nrm 以及 cnpm ，都是一个类似的过程。

## hexo安装配置

全局安装hexo

```powershell
npm install -g hexo-cli
```

