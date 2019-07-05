###  git教程



#### git工作原理

![图片描述](https://img.mukewang.com/59c31e4400013bc911720340.png "工作")

> * Workspace：工作区
> * Index / Stage：暂存区
> * Repository：仓库区（或本地仓库）
> * Remote：远程仓库

#### SVN与Git的最主要的区别

>SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而干活的时候，用的都是自己的电脑，所以首先要从中央服务器哪里得到最新的版本，然后干活，干完后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，如果在局域网还可以，带宽够大，速度够快，如果在互联网下，如果网速慢的话，就纳闷了。

> Git是分布式版本控制系统，那么它就没有中央服务器的，每个人的电脑就是一个完整的版本库，这样，工作的时候就不需要联网了，因为版本都是在自己的电脑上。既然每个人的电脑都有一个完整的版本库，那多个人如何协作呢？比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。

####  Git常用命令

1. 创建版本库

   ```
   git init
   echo "## new add text" >> README
   git status
   git add README
   git commit -m "mod README"
   ```

   

2. 版本回退

   ```
   git log --pretty=oneline (版本号 注释)
   git reflog 可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作）
   
   git reset --hard HEAD^ git reset --hard HEAD~100 git reset --hard 8807
   
   ```

3. 撤销删除

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
   
      