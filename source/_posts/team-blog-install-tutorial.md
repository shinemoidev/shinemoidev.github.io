---
title: Team blog install tutorial
date: 2018-01-22 09:53:03
tags: 其他
---
team-blog安装写作部署基本流程，初期可能需要适应一下，学几个新的命令，熟练了之后写起来还是蛮舒服的

<!--more-->

### 本机安装Hexo

参考[hexo安装教程](https://hexo.io/zh-cn/docs/)

### 去coding.net注册自己的账号，并添加ssh-key

注册完成后，彩云 @徐金良 告诉我你的用户名

我会把你加到blog项目里

添加成功后再进行以下步骤

### clone blog repo

```bash
$ git clone git@git.coding.net:shinemo_ios/shinemo_ios.git ~/iOS-team-blog
```

### to blog dir

```bash
$ cd ~/iOS-team-blog
```

### 安装依赖

#####注意不要用sudo安装, 否则可能会引起其他问题.

```bash
$ npm install hexo-deployer-git --save
$ npm install hexo-renderer-pug --save
$ npm install hexo-renderer-sass --save
```

### 错误处理

#### 错误1
```bash
In file included from ../src/binding.cpp:3:
../src/sass_context_wrapper.h:8:10: fatal error: 'sass/context.h' file not found
```
执行 brew install libsass 后重试

#### 错误2
```bash
dyld: lazy symbol binding failed: Symbol not found: _sass_make_boolean
```
执行 npm install node-sass 后重试

#### 错误3
```bash
rm: ./Release/.deps/Release/obj.target/fse/fsevents.o.d.raw: No such file or directory
make: *** [Release/obj.target/fse/fsevents.o] Error 1
```
执行 npm install fsevents 后重试.












### 写文章

```bash
$ hexo n "title"
```

写文章使用markdown格式，参考[markdown教程](https://www.jianshu.com/p/q81RER)

### 查看文章效果

#### 本地启动hexo server

```bash
$ hexo s
```

确认文章效果符合自己的预期

### 提交代码

```bash
$ git commit -s
```

```bash
$ git push origin master
```

### 部署blog

```bash
hexo g
```

```bash
hexo d
```

### 查看最终效果

浏览器打开[zxios.com](http://zxios.com)查看
