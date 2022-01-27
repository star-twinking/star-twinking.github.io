# Git rebase
git - rebase来自官方的解释是：Reapply commits on top of another base tip。中文两字就能概括：变基。
使用生活中的例子，就像接线头一样，将一段线条接在了另一个线条后面，最后它们依然是一条线。

## rebase 更新本地分支
假如你在本地master分支中已经开发了部分功能，产生了一些提交。
与此同时，你的小伙伴也努力工作，并将他的劳动成果推送到了远端分支origin/master上。此时，你需要将远端分支更新到本地，之后你才可以继续推送(push)，
分支情况如下图所示：
![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/basic-rebase-1.png "分叉的提交历史")

通常，我们的做法是执行git pull命令，该命令会自动下载分支并完成合并。
但git pull命令默认采用的合并策略是merge方法，即git pull是如下两条命令的缩写：
```
git fetch origin 
git merge origin master
```
执行后结果如下所示:
![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/basic-rebase-2.png "通过合并操作来整合分叉的历史")

如果使用rebase,就是相当于图中的c4 experiment 分支，然后将它变基到 master 分支上：
```
$ git rebase master experiment
```
如下图所示,它的原理是首先找到这两个分支（即**当前分支 experiment**、变基操作的**目标基底分支 master**） 的**最近共同祖先 C2**，然后对比当前分支相对于该祖先的历次提交，
提取相应的修改并存为临时文件， 然后将当前分支指向目标commit id:C3, 最后以此将之前另存为临时文件的修改依序应用。

$ git rebase <basebranch> <topicbranch> 的指令. Basebranch 是要整合进去的分支,这里是master, topicbranch是被整合的分支,
这里是experiment.

![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/basic-rebase-3.png "通过rebase操作来整合分叉的历史")

前面我们介绍过,comment id 指的是C0,C1,C2,C3,C4等指针地址.直观上看,rebase操作会把C4-C2的连接直接掐断,然后将C4'接到C3上,但是这个C4'是重新生成的
和原来的C4提交内容是一样的,但是commit ID是不一样的. 所以变基(rebase)会重新改变commit ID的地址,为原提交的内容重新分配commit ID,并将新的提交连接在
新的变基点上.

现在回到master分支上,对其进行fast-forward合并(前面提到过):
```
$ git checkout master
$ git merge experiment
```
最后结果如下所示:
![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/basic-rebase-4.png "master 分支的快进合并")

根据rebase的特性,我们还可以合并commit以及删除commit的操作,
具体参考[温故知新之git rebase的用法](https://juejin.cn/post/6844904089722208270)

## 更复杂的变基:分支里的分支变基

![](/images/Git_learn/rebase/interesting-rebase-1.png "Figure 1 从一个主题分支里再分出一个主题分支的提交历史")
如上图所示,现在master分支为主要分支,你将稳定的项目文件放在这个分支中.假设你需要给这个项目添加一个新的功能.在master分支C2提交之后,
开了一个server分支,在commit id为C3的提交中,你又想到了需要给server添加另外一个功能,但是不想影响server的开发,
就又在C3的提交之后给server添加了一个分支client.现在的情况就是分支中又开了一个分支.

![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/interesting-rebase-2.png "Figure 2 截取主题分支上的另一个主题分支，然后变基到其他分支")
如果client修改好了,你想先把在server的分支上的client这个功能先加到master上,但是不想server影响.
可以用:

```
$ git rebase --onto master server client
```
这个指令表示 将server分支下的client 变基(rebase)到 master上.
( “Take the client branch, figure out the patches since it diverged from the server branch, and replay these patches in 
the client branch as if it was based directly off the master branch instead.”)
最后的结果如上图2所示.commit id 为C8,C9的重新分配了C8’, C9’的地址.最后连接到了master上.分支server本身是不变的.

![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/interesting-rebase-3.png "Figure 3 快进合并 master 分支，使之包含来自 client 分支的修改")
现在想要将client合并,切换到master分支,用merge合并:

```
$ git checkout master
$ git merge client
```
合并后结果如上图3所示

![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/interesting-rebase-4.png "Figure 4 将 server 中的修改变基到 master 上")
接下来,如果要将server分支整合进来,可以使用 git rebase <basebranch> <topicbranch> 的指令. Basebranch 是要整合进去的分支,这里是master, 
topicbranch是被整合的分支,这里是 server.

```
$ git rebase master client
```
如图将 server 中的修改变基到 master 上所示，server 中的代码被“续”到了 master 后面。

![](https://raw.githubusercontent.com/star-twinking/CloudImage/main/ImgforBlog/interesting-rebase-5.png "Figure 5 最终提交历史")
然后就可以使用fast forward合并到主分支master,至此，client 和 server 分支中的修改都已经整合到主分支里了， 你可以删除这两个分支，
最终提交历史会变成图最终的提交历史 中的样子,如上图所示.

```
$ git checkout master
$ git merge server

# Delete two branches
$ git branch -d client
$ git branch -d server
```



## rebase命令的风险
变基过程可以让你以线性方式来管理你的提交，但rebase命令会修改你的提交id，说白了，rebase改变了历史，改变了过去，
“严谨的历史学家”可不能容忍随意的更改历史呀，所以它很危险。
reabse命令魅力无穷，使用得当它会使你的提交脉络清晰明了，使用不当你会陷入反复解决冲突的漩涡里。
为避免错误使用rebase命令，在这里引出一个使用rebase命令的“金科玉律”：

**永远不要去rebase本地之外的任何提交!**

**Do not rebase commits that exist outside your repository and that people may have based work on.**

意思是,**如果发现你操作的那些commit,已经被推送到远端或者被其他小伙伴合并到本地，请停下变基的操作。**
只要遵守这条定律，使用rebase命令修改私有分支的历史记录就不会出差错。

    主要是因为rebase会改变过去的commit,你的伙伴可不愿意你随意篡改他的劳动成果