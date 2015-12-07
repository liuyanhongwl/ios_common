### nvm 安装配置

[参考: 快速搭建 Node.js 开发环境以及加速 npm](!https://cnodejs.org/topic/5338c5db7cbade005b023c98)
#### 1. git clone nvm

```
cd ~/git
git clone https://github.com/creationix/nvm.git
```

#### 2. 配置nvm

让终端加载nvm命令
```
source ~/git/nvm/nvm.sh
```

配置终端启动时自动执行 

在 ~/.bashrc, ~/.bash_profile, ~/.profile, 或者 ~/.zshrc 文件添加`source`命令:

在终端输入 nvm

```
$ nvm

Node Version Manager

Note: <version> refers to any version-like string nvm understands. This includes:
  - full or partial version numbers, starting with an optional "v" (0.10, v0.1.2, v1)
  - default (built-in) aliases: node, stable, unstable, iojs, system
  - custom aliases you define with `nvm alias foo`

Usage:
  nvm help                                  Show this message
  nvm --version                             Print out the latest released version of nvm
  nvm install [-s] <version>                Download and install a <version>, [-s] from source. Uses .nvmrc if available
    --reinstall-packages-from=<version>     When installing, reinstall packages installed in <node|iojs|node version number>
  nvm uninstall <version>                   Uninstall a version
  nvm use [--silent] <version>              Modify PATH to use <version>. Uses .nvmrc if available
  nvm exec [--silent] <version> [<command>] Run <command> on <version>. Uses .nvmrc if available
  nvm run [--silent] <version> [<args>]     Run `node` on <version> with <args> as arguments. Uses .nvmrc if available
  nvm current                               Display currently activated version
  nvm ls                                    List installed versions
  nvm ls <version>                          List versions matching a given description
  nvm ls-remote                             List remote versions available for install
  nvm version <version>                     Resolve the given description to a single local version
  nvm version-remote <version>              Resolve the given description to a single remote version
  nvm deactivate                            Undo effects of `nvm` on current shell
  nvm alias [<pattern>]                     Show all aliases beginning with <pattern>
  nvm alias <name> <version>                Set an alias named <name> pointing to <version>
  nvm unalias <name>                        Deletes the alias named <name>
  nvm reinstall-packages <version>          Reinstall global `npm` packages contained in <version> to current version
  nvm unload                                Unload `nvm` from shell
  nvm which [<version>]                     Display path to installed node version. Uses .nvmrc if available

Example:
  nvm install v0.10.32                  Install a specific version number
  nvm use 0.10                          Use the latest available 0.10.x release
  nvm run 0.10.32 app.js                Run app.js using node v0.10.32
  nvm exec 0.10.32 node app.js          Run `node app.js` with the PATH pointing to node v0.10.32
  nvm alias default 0.10.32             Set default node version on a shell

Note:
  to remove, delete, or uninstall nvm - just remove the `$NVM_DIR` folder (usually `~/.nvm`)
```

#### 3. 通过nvm安装任意版本node

上面nvm命令文档中有Example:

```
Example:
  nvm install v0.10.32                  Install a specific version number
  nvm use 0.10                          Use the latest available 0.10.x release
  nvm run 0.10.32 app.js                Run app.js using node v0.10.32
  nvm exec 0.10.32 node app.js          Run `node app.js` with the PATH pointing to node v0.10.32
  nvm alias default 0.10.32             Set default node version on a shell
```

nvm 默认是从 http://nodejs.org/dist/ 下载的, 国外服务器, 必然很慢,好在 nvm 以及支持从镜像服务器下载包, 于是我们可以方便地从七牛的 node dist 镜像下载:

```
$ NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/dist nvm install 0.11.11
```
于是你就会看到一段非常快速进度条:

```
######################################################################## 100.0%
Now using node v0.11.11
```

如果你不想每次都输入环境变量 NVM_NODEJS_ORG_MIRROR, 那么我建议你加入到 ~/.bashrc, ~/.bash_profile, ~/.profile, 或者 ~/.zshrc 文件中:

```
# nvm
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/dist
source ~/git/nvm/nvm.sh
```

然后你可以继续非常方便地安装各个版本的 node 了, 你可以查看一下你当前已经安装的版本:

```
$ nvm ls
       v0.11.11
         v5.0.0
node -> stable (-> v5.0.0) (default)
stable -> 5.0 (-> v5.0.0) (default)
unstable -> 0.11 (-> v0.11.11) (default)
iojs -> N/A (default)
```
















