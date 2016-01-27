## Mac下终端使用Github


 - ##### 打开终端，先测试一下你的帐号跟github连上没有

<pre><code>ssh -T git@github.com</code></pre>

<code> Hi XXXX! You've successfully authenticated, but GitHub does not provide shell access. </code> 如果出现这个提示，表示你连已经连上了.

 - ##### 在github建自己的Repository。
 
 建好后下面有一些代码提示你如何同步，当然先要cd到自己需要同步的目录，然后代码如下：
 
 
 <pre><code>1. touch README.md //新建一个记录提交操作的文档  
2. git init //初始化本地仓库  
3. git add README.md //添加  
4. git commit -m "first commit"//提交到要地仓库，并写一些注释  
5. git remote add origin git@github.com:youname/Test.git //连接远程仓库并建了一个名叫：origin的别名  
6. git push -u origin master //将本地仓库的东西提交到地址是origin的地址，master分支下  </code></pre>

基本上就这么一个过程了，如果你的本地址仓库还有很多其它东西要提交，先要在本地仓库commit， 方法也是先git add 然后 git commit，这些都是commit到本地仓库，然后再push到选程.

下一次操作时，名叫origin的远程地址上次已经建好了，想看看可以用git remote -v 查看地址。
push的时候，先pull一下，本地同步下，再push

------
#### 检出仓库

- 执行如下命令以创建一个本地仓库的克隆版本：
git clone /path/to/repository

- 如果是远端服务器上的仓库，你的命令会是这个样子：
git clone username@host:/path/to/repository

#### 工作流

你的本地仓库由 git 维护的三棵“树”组成。第一个是你的 工作目录，它持有实际文件；第二个是 缓存区（Index），它像个缓存区域，临时保存你的改动；最后是 HEAD，指向你最近一次提交后的结果。

#### 添加与提交

- 你可以计划改动（把它们添加到缓存区），使用如下命令：    
 git add <filename>    
 git add *    
这是 git 基本工作流程的第一步；
- 使用如下命令以实际提交改动：
git commit -m "代码提交信息"

现在，你的改动已经提交到了 HEAD，但是还没到你的远端仓库。

#### 推送改动

- 你的改动现在已经在本地仓库的 HEAD 中了。执行如下命令以将这些改动提交到远端仓库：
git push origin master
可以把 master 换成你想要推送的任何分支。

- 如果你还没有克隆现有仓库，并欲将你的仓库连接到某个远程服务器，你可以使用如下命令添加：
git remote add origin <server>

如此你就能够将你的改动推送到所添加的服务器上去了。

#### 分支

- 创建一个叫做“feature_x”的分支，并切换过去：
git checkout -b feature_x

- 切换回主分支：
git checkout master

- 再把新建的分支删掉：
git branch -d feature_x

- 除非你将分支推送到远端仓库，不然该分支就是 不为他人所见的：
git push origin <branch>

#### 更新与合并

- 要更新你的本地仓库至最新改动，执行：
git pull    
以在你的工作目录中 获取（fetch） 并 合并（merge） 远端的改动。

- 要合并其他分支到你的当前分支（例如 master），执行：
git merge <branch>

两种情况下，git 都会尝试去自动合并改动。不幸的是，自动合并并非次次都能成功，并可能导致 冲突（conflicts）。 这时候就需要你修改这些文件来人肉合并这些 冲突（conflicts） 了。改完之后，你需要执行如下命令以将它们标记为合并成功：
git add <filename>

在合并改动之前，也可以使用如下命令查看：
git diff <source_branch> <target_branch>

**有个主分支master, 两个分支branch1和branch2, 合并时：**

##### 1. merge合并

执行fast-forward（快进）合并。

* 切换到master，合并branch1  

```
git checkout master
git merge branch1
```
* 接着合并branch2

```
git merge branch2
```

* 上步骤可能会有冲突，解决冲突后，重新提交

```
git add .
git commit -a -m 'commit test'

```



##### 2. rebase合并

合并branch2分支的时候，使用rebase可以使提交的历史记录显得更简洁。

* 取消刚才的合并（上步骤用 merge 合并branch2）

```
$ git reset --hard HEAD~
```

* 切换到issue3分支后，对master执行rebase。

```
$ git checkout branch2
$ git rebase master

```
* 和merge时的操作相同，修改在发生冲突的部分。

* rebase的时候，修改冲突后的提交不是使用commit命令，而是执行rebase命令指定 --continue选项。若要取消rebase，指定 --abort选项。

```
$ git add .
$ git rebase --continue
```

* 这样，在master分支的issue3分支就可以fast-forward合并了。切换到master分支后执行合并。

```
$ git checkout master
$ git merge branch2
```



#### 替换本地改动

假如你做错事（自然，这是不可能的），你可以使用如下命令替换掉本地改动：
git checkout -- <filename>
此命令会使用 HEAD 中的最新内容替换掉你的工作目录中的文件。已添加到缓存区的改动，以及新文件，都不受影响。

假如你想要丢弃你所有的本地改动与提交，可以到服务器上获取最新的版本并将你本地主分支指向到它：
git fetch origin   
git reset --hard origin/master

-----
### 常用命令
- 拉取 git pull


- 新建分支   
git checkout -b feature-xxx master   表示从 master 新建一个 feature-xxx 新分支   
git checkout -b feature-yyy feature-xxx


- 提交修改    
git add -A .  	把所有的修改加入 commit    
git commit -a -m ‘1.增加了xxx；2. 修改了yyy。’     新增一个 commit     
git push origin master    将修改提交到服务器的 feature-ui


- git status # 查看本地仓库的修改状态     
git add # 暂存文件     
git commit # 提交文件     
git stash # 保存当前修改    
git stash list #保存的修改站     
git stash pop #拿出保存的修改    

用 .gitignore文件 来避免把一些不必要的文件上传到 GitLab，内容如下，   
.DS_Store    
*xcuserdata*    
*.a    
*.framework    
build/    
Podfile.lock    
Pods/    

.gitignore 是可以影响子目录。