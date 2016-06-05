---
title: github pages ＋ hexo ＋ next搭建个人博客
date: 2016-05-04 21:07:46
tags: hexo
---

## 概述

github pages原本是github提供给开源项目做项目介绍的，既然是静态页面，当然拿来做blog既免费又方便了，现在流行用hexo框架 ＋ next主题，鉴于本人在搭建过程中踩过的坑，这里记录一下，希望能够方便后来人。

<!-- more -->

## 环境配置

### Git
Mac自带，windows需要下载msysgit。为了后期方便，最好是配置好ssh等信息。

首先在github创建项目`bobomee.github.io`，这里仓库名规则是一定的。

### Node.js
mac下使用brew命令安装
```java
// 安装 node
brew install node
```
如果出现错误，尝试使用如下命令
```java
sudo chown -R $(whoami):admin /usr/local
```

查看是否安装成功
```java
node -v
```

### hexo

使用npm来安装hexo
```java
// npm在git bash中使用
sudo npm install -g hexo
```
这里有坑，官网命令：`npm install -g hexo-cli`会出现权限错误。

查看是否安装成功
```java
hexo -v
```

## 初始化blog
### hexo init
新建blog目录，为方便操作，在桌面上新建目录
```java
ls
cd Desktop/
mkdir hexo
cd hexo/
hexo init
npm install
```

开启本地服务预览
```java
hexo s
```
此命令执行后，就可以通过`http://localhost:4000`查看博客了。

### config blog

此时会看到hexo目录下文件

```java
_config.yml    
db.json
node_modules
package.json
scaffolds
source
themes
```
_config.yml 是全局配置文件，在最后加上如下语句和github关联起来

```java
deploy:
    type: git
    repository: https://github.com/BoBoMEe/bobomee.github.io.git
    branch: master
```

这里有坑：需要注意冒号后面需要有空格，其实这个文件的所有冒号后面都需要加上空格。

### deploy

推送到github，可以用如下命令

```java
hexo g //generate
hexo d //deploy
```
这里有坑：如果更改后需要先`hexo clean`之后再重新生成网页,同时修改了_config.yml中的deploy信息，需要先调用`npm install hexo-deployer-git --save`


## 主题设置

使用hexo比较流行的next主题，配置可见[hexo-theme-next](https://github.com/iissnan/hexo-theme-next)

##  源码托管

如果我们不在一个地方写blog的话，可以尝试将blog的source也托管到分支上

```java
ls
cd Desktop/
cd hexo/
git init
git checkout --orphan gh-pages
git add .
git commit -m "blog source"
git remote add origin https://github.com/BoBoMEe/bobomee.github.io.git
git push origin gh-pages
```
## 发表文章

```java
hexo new "postName"
```
会在`/blog/source/_posts`生成postName.md,也可以直接新建。

## 常用命令

```java
hexo new "这是一篇新文章"// 新建文档
hexo clean//清空缓存
hexo g// 生成静态页面
hexo s// 预览, 可打开浏览器进行预览
hexo d// 发布到 github ，需在站点配置文件中配置正确
```

更多hexo和thene配置参考：

[hexo配置](http://www.jianshu.com/p/858ecf233db9)

[hexo-theme-next](https://github.com/iissnan/hexo-theme-next)
