---
title: 利用sinopia搭建npm私服
date: 2017-05-14 09:16:11
tags: npm
categories: npm
---

相信大家在合作开发的时候，或者项目稍微大一些的时候，总会遇到npm各种各样的问题，可能慢，可能卡，可能失败

项目大的时候，你写的组件也可能不只是自己用，别的开发也需要统一用，所以一遍一遍的copy，不如一了百了把上面的问题也一起解决了

我所在项目是因为有三个系统，最后要合并到一起，再一个是项目的保密性，在公司内局域网的时候，无法连接外部网络，所以npm无法正常使用。

好了，废话不多说了，开始npm私服搭建：

首先你需要准备好相应的环境

* node环境 *（去网上下载好）*
* nrm *(npm 源管理器，这个非常方便 $ npm install -g nrm # 安装nrm)*
* sinopia *一个非常方便的搭建npm私服的工具（npm install -g sinopia）*

>$ sinopia
>
>warn  --- config file  - .....\AppData\Roaming\sinopia\config.yaml
>
>warn  --- http address - http://localhost:4873/
然后打开：http://localhost:4873/

你需要找到sinopia在你电脑上的安装目录，然后会发现目录下默认有两个文件：config.yaml和storage ，htpasswd 是添加用户之后自动创建的。

config.yaml就是配置文件啦

```
#
# This is the default config file. It allows all users to do anything,
# so don't use it on production systems.
#
# Look here for more config file examples:
# https://github.com/rlidwka/sinopia/tree/master/conf
#

# path to a directory with all packages
storage: ./storage  //npm包存放的路径

auth:
  htpasswd:
    file: ./htpasswd   //保存用户的账号密码等信息
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    max_users: -1  //默认为1000，改为-1，禁止注册

# a list of other known repositories we can talk to
uplinks:
  npmjs:
    url: http://registry.npm.taobao.org/  //默认为npm的官网
    
packages:  //配置权限管理
  '@*/*':
    # scoped packages
    access: $all
    publish: $authenticated

  '*':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs

# log settings
logs:
  - {type: stdout, format: pretty, level: http}
  #- {type: file, path: sinopia.log, level: info}

# you can specify listen address (or simply a port) 
listen: 0.0.0.0:4873  ////默认没有，只能在本机访问，添加后可以通过外网访问。（换为本机IP即可）

```


## 部分配置字段意义

<font color="red">**storage**</font> :仓库保存的路径

<font color="red">**web**</font> : 是否支持WEB接口

<font color="red">**auth**</font> : 验证相关

<font color="red">**uplinks**</font> : 配置上游的npm服务器，主要是用于请求的仓库不存在时去上游服务器拉取

<font color="red">**packages**</font> : 配置模块/包的发布(publish)、下载(access)的权限等

<font color="red">**listen**</font> : 配置监听端口与主机名

## packages配置

配置大致分为两个部分，一个是以 @weflex/* 为开头的，另一个则是通配符 *。

这个当然就是对 package.json 中的 name 字段进行匹配，比如 @weflex/app 将匹配第一个配置，而 express 则匹配第二个。

这里这么配置的意义在于：一般团队或者公司的私有项目，会采用不同的权限控制，于是这里借用了 NPM 的 scoped name 即 @company 的形式，例如 @weflex/app 即表示 WeFlex 下属的 app 项目了。

接下来，每一个命名过滤器(filter)下都有三项基本设置：

* access: 表示哪一类用户可以对匹配的项目进行安装(install)
* publish: 表示哪一类用户可以对匹配的项目进行发布(publish)
* proxy: 如其名，这里的值是对应于 uplinks 的

对于1和2的值，我们通常有以下一些可选的配置：

* $all 表示所有人都可以执行对应的操作
* $authenticated 表示只有通过验证的人可以执行对应操作
* $anonymous 表示只有匿名者可以进行对应操作（通常无用）

或者也可以指定对应于之前我们配置的用户表 htpasswd 中的一个或多个用户，这样就明确地指定哪些用户可以执行匹配的操作，再重启就可以了

## 客户端使用
强烈推荐使用nrm来管理自己的代理。
>$ nrm add <name> http://XXXXXX:4873 # 添加本地的npm镜像地址
>
> $ nrm use <name> # 使用本址的镜像地址 
     
>

nrm的其他命令：
>$ nrm --help  # 查看nrm命令帮助
>
>    $ nrm list # 列出可用的 npm 镜像地址
>
>   $ nrm use taobao # 使用`淘宝npm`镜像地址


## 创建用户与发布包

创建新用户

1.确保自己已经切换到配置的代理

>  $ nrm ls
>pm ---- https://registry.npmjs.org/
>
>    cnpm --- http://r.cnpmjs.org/
>
>    taobao - http://registry.npm.taobao.org/
>
>    nj ----- https://registry.nodejitsu.com/
>
>    rednpm - http://registry.mirror.cqupt.edu.cn
>
>    npmMirror  https://skimdb.npmjs.com/registry
>
>    *&nbsp;sinopia  http://192.168.1.200:4873/

2.运行npm adduser,填写信息，注册账号。如果已经有账号，直接运行npm login即可。(<font color="red">*这时候一定要确定自己的npm的源在私服*</font>)

$ npm adduser
>Username: test
>
>    Password:
>
>    Email(this IS public)


3.创建自己的组件

4.在组件的根目录下进行 $ npm init ， 然后填写相关的信息就可以

5.目录向上一层，运行$ npm publish \<packageName\> 发布新包。