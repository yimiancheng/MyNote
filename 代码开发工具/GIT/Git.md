#### git工作原理

![图片描述](https://img.mukewang.com/59c31e4400013bc911720340.png "工作")

> * Workspace：工作区
> * Index / Stage：暂存区
> * Repository：仓库区（或本地仓库）
> * Remote：远程仓库

#### SVN与Git的最主要的区别

>SVN是集中式版本控制系统，版本库是集中放在中央服务re器的，而干活的时候，用的都是自己的电脑，所以首先要从中央服务器哪里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，如果在局域网还可以，带宽够大，速度够快，如果在互联网下，如果网速慢的话，就纳闷了。

> Git是分布式版本控制系统，那么它就没有中央服务器的，每个人的电脑就是一个完整的版本库，这样，工作的时候就不需要联网了，因为版本都是在自己的电脑上。既然每个人的电脑都有一个完整的版本库，那多个人如何协作呢？比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

#### Git安装

[git官网](https://git-scm.com/)

*TortoiseGit-2.5.0.0-64bit.msi*

> ```
> git config --list
> 添加全局配置 git config --global user.name  "xxx" / git config --global user.email "xxx@126.com"
> 本地的工程里面添加单独的配置 git config --local  user.name  "xxx"
> 编辑git信息：git config -e --global / git config -e --local
> ```

####  Git常用命令

##### 1. 创建版本库

   ```shell
   git init / git init MyNote
   echo "## new add text" >> README
   git status
   git add README / git add [file1] [file2] ... / git add [dir]
   git commit -m "mod README"
   git 
   git diff master origin/master
   ```
   [git常用命令](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
   ```
# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p
 
# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...
 
# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]
 
# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区 可省略 git add
$ git commit -a

# 当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名 git pull origin
# 当前分支只有一个追踪分支，连远程主机名都可以省略 git pull
git pull origin master
   ```

##### 2. 版本回退

   ```
   git log --pretty=oneline (版本号 注释)
   git reflog 可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作）
   
   git reset --hard HEAD^ git reset --hard HEAD~100 git reset --hard 8807
   
   ```

##### 3. 撤销删除

   * 修改了工作区的文件，还未提交到暂存区

   > **误删了工作区的某个文件，可以从仓库里恢复**

      ```
   git checkout -- README (--很重要，没有--，就变成了“切换到另一个分支”的命令)
      ```

   * 修改了文件，还执行了git add 到暂存区
   
      ```
      git reset HEAD README (把README暂存区修改的内容撤消到工作区)
      git checkout -- README
      ```

##### 4. 分支操作

```
查看分支：git branch / git branch -a / git branch -v / git branch -r
创建分支：git branch dev
创建+切换分支：git checkout -b dev
切换分支：git checkout dev
删除分支：git branch -d dev
合并某分支到当前分支：git merge dev

提交该分支到远程仓库: git push origin dev
建立本地到上游（远端）仓的链接: git branch --set-upstream-to=origin/dev 
取消对master的跟踪: git branch --unset-upstream master
删除远程分支: git push origin --delete dev 
```

> Fast-forward 快进模式 master指向dev的当前提交 **合并速度非常快**
>
> 创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在master分支上工作效果是一样的，但过程更安全

##### 5. 合并时提示冲突 

![Git版本工具的部署与使用](E:\MyNote\assets\67ba796e0dd182c5850b433456429c80.png)

```

<<<<<<< HEAD
aa simple
=======
test simple
>>>>>>> feature1
```

Git用`<<<<<<<HEAD`来表示是主分支修改的内容，`=======`表示分隔符，`>>>>>>>`表示是feature1分支修改的内容。

需要人工重新修正README文件，然后再**提交**才能解决冲突

![Git版本工具的部署与使用](E:\MyNote\assets\c541ac235bb47f489a447e6f9e39b7f4.png)

```
graph=graph abbrev=缩写 pretty=漂亮
git log --graph --pretty=oneline --abbrev-commit
*   ecea0d4 conflict fixed
|\  
| * 594482c test simple
* | 34db356 aa simple
|/  
*   2452222 add test
```

* 禁用Fast forward模式管理

   >合并分支时，Git默认使用用Fast forward模式，删除分支后，分支信息 **会永久删除**
   >
   >如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，从分支历史上就可以看出分支信息。
   >
   >使用  **--no-ff ** 参数来禁用Fast forward模式
   >
   >git merge --no-ff -m "merge with no-ff" dev    -当禁用Fast forward模式时，合并就是一个新的commit，所以加上-m参数
   >
   >```
   >git log --graph --pretty=oneline --abbrev-commit 
   >*   7668b3d merge with no-ff
   >|\  
   >| * 93df64b add merge
   >|/  
   >*   ecea0d4 conflict fixed
   >```

   ![img](E:\MyNote\assets\d4ff0ed4f4117c5c6a5fd22eb04fe9ba.png)

##### 6. git stash

> 工作进行到一半时候，我们还无法提交，只能 add, 可以把当前工作现场 `隐藏起来`，等以后恢复现场后继续工作

```
git stash 当前工作现场异常，git status是干净的
git stash list

恢复：
git stash apply stash内容并不删除，需要使用命令git stash drop来删除
git stash pop 恢复的同时把stash内容也删除了
git stash drop 只删除第一条
```
##### 7. git多人写作

> 当你从远程库克隆时候，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且远程库的默认名称是origin。
>
> 要查看远程库的信息 使用 git remote
> 要查看远程库的详细信息 使用 git remote –v



```
创建dev分支：git checkout –b dev origin/dev 把远程的origin的dev分支到本地来

git branch --set-upstream-to=origin/dev 指定本地dev分支与远程origin/dev分支的链接
git pull
```

##### 8. git tag
> Git 也可以对某一时间点上的版本打上标签。人们在发布某个软件版本（比如 v1.0 等等）的时候

```
列显已有的标签: git tag / git tag -l 'v1.4.2.*'
新建含附注的标签：git tag -a v1.4 -m 'my version 1.4'
```

####  Git .gitignore

> 在Git工作区的根目录下创建一个特殊的`.gitignore`文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件
>
> 定义Git全局的 .gitignore 文件: git config --global core.excludesfile ~/.gitignore


##### 忽略规则的语法:

- bin/: 忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件

- /bin: 忽略根目录下的bin文件

- /*.c: 忽略 cat.c，不忽略 build/cat.c

- debug/*.obj: 忽略 debug/io.obj，不忽略 debug/common/io.obj 和 tools/debug/io.obj

- **/foo: 忽略/foo, a/foo, a/b/foo等

- a/**/b: 忽略a/b, a/x/b, a/x/y/b等

- !/bin/run.sh: 不忽略 bin 目录下的 run.sh 文件

- *.log: 忽略所有 .log 文件

- config.php: 忽略当前路径的 config.php 文件

- *.py[cod]: 忽略所有 .pyc、.pyo 文件


```
git rm -r --cached . 先把本地暂存区删除
git add .
```




#### 参考教程

[csdn-git使用教程](https://blog.csdn.net/qq_36150631/article/details/81038485 "csdn-git使用教程")

[csdn-git部署](https://blog.csdn.net/hanyuyang19940104/article/details/80194601 "git部署")