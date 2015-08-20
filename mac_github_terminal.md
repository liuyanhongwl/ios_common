## Mac下终端使用Github


 - ##### 打开终端，先测试一下你的帐号跟github连上没有

<pre><code>ssh -T git@github.com</code></pre>

<code> Hi XXXX! You've successfully authenticated, but GitHub does not provide shell access. </code> 如果出现这个提示，表示你连已经连上了.

 - ###### 在github建自己的Repository。
 
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