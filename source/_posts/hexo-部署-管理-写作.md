---
title: hexo-部署-管理-写作
tags:
  - hexo
categories:
  - 博客
date: 2020-08-26 17:06:09
---


## hexo建站

建站有两个前提：

* Node安装
* Git安装

### node安装

在windows上管理node的多版本安装，可以使用[Node Version Manager for Windows](https://github.com/coreybutler/nvm-windows "nvm")

安装[nvm](https://github.com/coreybutler/nvm-windows)之前，需要卸载所有已存在的node以及所有已存在的npm

#### 卸载node

控制面板删除node以后，还需要删除node的安装文件夹比如（C:\Program Files\node.js)

#### 卸载npm

删除已存在的npm安装位置，比如（`C::\Users\user\AppData\Roaming\npm\ect\npmrc`）。
备份全局`npmrc` 配置，比如（`C:\Users\<user>\AppData\Roaming\npm\etc\npmrc`)。如果需要用到这些配置文件，拷贝到（`C:\Users\user\.npmrc`）

#### 更新nvm

重新运行nvm的新版本安装包即可

#### 使用nvm安装管理node

介绍几个nvm的常用命令，以便用来安装node，更详细的命令说明，查看[nvm的github page](https://github.com/coreybutler/nvm-windows)

1. `nvm list [available]`查看已安装的node版本，带上`available`参数则显示可以安装的node版本
2. `nvm on`启用node版本管理
3. `nvm off`关闭node版本管理
4. `nvm install <version> [arch]`安装指定的node版本，`arch`参数指定安装的版本是32位或者64位
5. `nvm uninstall <version>` 卸载指定的版本
6. `nvm use <version> [arch]` 切换到已安装的指定版本
7. `nvm node_mirror <node_mirro_url>`设置node的镜像源，淘宝的镜像源<https://npm.taobao.org/mirrors/node/>
8. `nvm npm_mirror <npm_mirror_url>`设置npm的镜像源，淘宝的镜像源*https://npm.taobao.org/mirrors/npm/*

### Git安装与配置

下载对应的[git](https://git-scm.com/)版本安装即可

#### 配置

github的ssh配置在[github doc](https://docs.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh)。

要在本地git上使用ssh连接github，需要在本地对git进行配置

1. 生成ssh密钥

2. 添加私钥到ssh-agent

   ```
   eval $(ssh-agent -s)
   
   ssh-add ~/.ssh/id_rsa
   ```

自动启动windows上的 `ssh-agent`复制以下行并将其粘贴到 Git shell 中的 `~/.profile` 或 `~/.bashrc` 文件中，通常情况下，目录所在位置为（C:\Users\user）：

```
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2= agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset env
```

如果私钥文件不是默认的私钥文件（`~/.ssh/id_rsa`)，则上文的脚本的`ssh-add`后需要添加自定义私钥文件位置

### hexo安装

```
$ npm install -g hexo-cli
```

或者

```
$ npm install hexo
```

#### 建站

```
$ hexo init <folder>
$ cd <folder>
$ npm install
```

#### 管理

hexo使用站点的master分支生成站点目录`public/`，在部署的时候会将`public/`站点目录推送至`_config.yml`中指定的远端仓库和分支。默认分支是`gh-pages`。

hexo每次部署的时候，会向目标仓库的对应分支下推送`public/`目录，并完全覆盖所有现有的内容。

所以在站点的管理中，站点目录的分支不能与其他分支特别是`mater`分支混淆。

在需要使用自定域名时，Github Pages需要有一个CNAME文件。此文件必须放置于`source`目录下，这样每次部署的时候，才能将CNAME文件一并推送。

##### 部署工具`hexo-deployer-git`安装

```
$ npm install hexo-deployer-git --save
```

##### 修改配置

```
deploy:
  type: git
  repo: <repository url> #https://bitbucket.org/JohnSmith/johnsmith.bitbucket.io
  branch: [branch]
  message: [message]
```

##### 配置参数说明

| 参数      | 描述                 | 默认                                                         |
| :-------- | :------------------- | :----------------------------------------------------------- |
| `repo`    | 库（Repository）地址 |                                                              |
| `branch`  | 分支名称             | `gh-pages` (GitHub) `coding-pages` (Coding.net) `master` (others) |
| `message` | 自定义提交信息       | `Site updated: {{ now('YYYY-MM-DD HH:mm:ss') }}`)            |

##### 生成站点文件并且部署

```
hexo clean && hexo deploy
```

##### 修改Githup Pages配置

在github的Repository Settings中，将默认分支改为_config.yml中的部署分支。

#### hexo从仓库重新生成站点

使用git clone将站点从远程仓库clone下来。切换到站点管理分支，默认是master。重新进行上文*安装-建站-管理* 的步骤即可。

