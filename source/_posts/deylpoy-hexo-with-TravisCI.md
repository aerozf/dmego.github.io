---
layout: post
title: "使用Travis CI自动部署Hexo到GitHub"
date: 2017-10-13 17:46
comments: true
categories: [博客]
tags: [hexo,Travis CI,自动部署]
copyright: true
---
## 前言
使用 `hexo + gitPages` 搭建个人博客的人都知道，每当要发表一篇博文，第一步得手动使用 `hexo g` 命令生成静态网页，然后还得通过 `hexo d` 命令将静态文件推送到GitHub远程仓库,不说麻烦不麻烦，更重要的是有时候环境换了，没有搭建 hexo 环境，想发篇博客的时候就没有可能了。而现在通过 Travis CI 就能自动构建自己的博客。而我们只需将写好的 `Markdown` 格式的博文`push` 到 源文件 分支即可。
## Travis CI 介绍
[Travis CI](https://travis-ci.org/) 是目前新兴的开源持续集成构建项目，它与 jenkins，GO的很明显的特别在于采用 yaml 格式，简洁清新独树一帜。目前大多数的 github 项目都已经移入到 Travis CI 的构建队列中，据说 Travis CI 每天运行超过 4000 次完整构建。
  <!--more-->
## hexo 介绍
[Hexo](https://hexo.io/) 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
## 使用 Travis 自动构建
我的博客自动部署思路是，将  `hexo` 源码 `push` 到博客 项目的另外一个分支，
既一个分支放源码，一个分支放静态文件，使用 `Travis CI` 自动部署 hexo 源码的分支，构建完成后自动推送到 静态文件的分支上，而这一切都在一个仓库上进行操作。<br>
注意：如果使用的是`GitPage`的个人站点来搭建博客的 ，则博客静态文件在 `master`分支上；如果使用的是 `gitPages` 的项目站点来搭建博客，则博客的静态文件在 `gh-pages` 分支上。
### 在GitHub 上生成 Access Token
如果想要 让`travis CI` 构建完成之后自动 push 到 master 分支，则travis需要有对这个仓库进行操作的权限，此时我们就需要为Travis CI 配置Access Token（访问令牌）。<br>
在GitHub上生成Access Token 的步骤是，点击头像进入设置（Settings）,r然后点击左边菜单栏最下面的`Developer settings` 选项，进入后点击左边的 `Personal access tokens` 选项，进入后点击右上角的`Generate new token` 按钮
![mark](http://ovasw3yf9.bkt.clouddn.com/blog/171014/G0hFA1LkK7.png?imageslim)
点击后就会来到下面的界面，先给 Token 起一个名字，然后为它设置一些权限，其中红框内的权限是必须的，其他可以随意添加。
![mark](http://ovasw3yf9.bkt.clouddn.com/blog/171014/5G22L5hCcK.png?imageslim)
点击下面的 `create token` 按钮，就会生成一个已经赋予好权限的 token 值，接下来我们Travis CI 网站的配置中。
![mark](http://ovasw3yf9.bkt.clouddn.com/blog/171014/fldkB30k3m.png?imageslim)

### 配置 Travis CI
如果之前从未使用 [Travis CI](https://travis-ci.org/) 来构建项目，则我们先需要使用GitHub账号来登录网站,登录进来后，会进到如下图界面，如果底下 没有把 GitHub 仓库中的项目加载进来，可以手动点击右上角的  `Sync account` 按钮，待到同步完成后将要自动构建的项目开启。
![mark](http://ovasw3yf9.bkt.clouddn.com/blog/171014/0IbbdiJh18.png?imageslim)
开启后点击设置图标就可以进行一系列的设置，如下图所示，先开启 `General` 里的两项选项：
- `Build only if .travis.yml is present`:只有在`.travis.yml`文件中配置的分支改变了才构建
- `Build branch updates`:当分支更新后开始构建

然后在  `Environment Variables` 一栏里将在 GitHub 下获取的的 `Access Token` 值添加进来
![mark](http://ovasw3yf9.bkt.clouddn.com/blog/171014/3b875iHdi4.png?imageslim)
### 添加配置文件到Hexo源码分支下
上面提到的 `.travis.yml` 配置文件需要添加到hexo 源码的根目录下，因为Travis CI 在自动构建时需要获取这些配置信息，以此来完成构建任务；这些配置信息主要包括源码分支，静态文件推送分支，仓库地址等信息。
![mark](http://ovasw3yf9.bkt.clouddn.com/blog/171014/CaBF4laGji.png?imageslim)
其中主要内容如下：
```
language: node_js
node_js: stable

# S: Build Lifecycle
install:
  - npm install


#before_script:
 # - npm install -g gulp

script:
  - hexo g

after_script:
  - cd ./public
  - git init
  - git config user.name "GitHub账户名称"
  - git config user.email "github账户邮箱"
  - git add .
  - git commit -m "Update docs"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
# E: Build LifeCycle

branches:
  only:
    - Hexo源码分支名称
env:
 global:
   - GH_REF: github.com/dmego/dmego.github.io.git(仓库地址)
```
配置到这一步就已经把所有配置全部完成，下面就是验证的过程

## 构建自动部署结果
将某篇文章中的一个表格增加一行后将修改推送到hexo源码所在的`hexo`分支
,然后等Travis CI 构建并自动部署完成后，点击博文发现表格多了一行。这说明构建自动部署项目成功。
![mark](http://ovasw3yf9.bkt.clouddn.com/blog/171014/hk2hCAma3D.png?imageslim)
