---
title: Team blog install tutorial
date: 2018-01-22 09:53:03
tags: 其他
---
team-blog安装写作部署基本流程，初期可能需要适应一下，学几个新的命令，熟练了之后写起来还是蛮舒服的

<!--more-->

### 本机安装Hexo

Hexo 需要 node.js 支持，所以要先安装 node.js，为了避免不必要的权限问题，推荐使用 nvm 安装 node.js

1. 安装 nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

2. 安装 node

```bash
nvm install node
```

3. 安装 hexo

```bash
npm install -g hexo-cli
```

参考[hexo安装教程](https://hexo.io/zh-cn/docs/)

### 去 github 注册自己的账号，并添加ssh-key

注册完成后，彩云 @徐金良 告诉我你的注册邮箱

我会把你加到blog项目里

添加成功后再进行以下步骤

### clone blog repo

```bash
git clone git@github.com:shinemoidev/shinemoidev.github.io.git ~/iOS-team-blog
```

### to blog dir

```bash
cd ~/iOS-team-blog
```

### 安装依赖

#####注意不要用sudo安装, 否则可能会引起其他问题.

```bash
npm install hexo-deployer-git --save
```

### 写文章

```bash
hexo n "title"
```

写文章使用markdown格式，参考[markdown教程](https://www.jianshu.com/p/q81RER)

### 查看文章效果

#### 本地启动hexo server

```bash
hexo g

hexo s
```

确认文章效果符合自己的预期

### 提交代码

```bash
git add .

git commit -s

git push origin main
```

### 部署blog

```bash
hexo g
```

此时若 git status 有变化，则需要做一次提交，提交完成之后再进行部署

```bash
hexo d
```

### 查看最终效果

浏览器打开[shinemoidev.github.io](https://shinemoidev.github.io/)查看

<!-- ### 错误处理

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
 -->

