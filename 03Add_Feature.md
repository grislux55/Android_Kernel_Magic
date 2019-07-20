# 三、内核功能添加

---

### 准备步骤
`本次教程使用的是Deepin，理论上Ubuntu和Debian都能用。另外，为了这个教程，我特地又装了一次Deepin，都给我泪目！`
1. `Linux`系统 x1（当然你想要更多系统也随你便）
2. `GitKraken`软件 x1 [下载地址](https://www.gitkraken.com/download)（因为它在Linux系统下表现较好，当然如果你要使用其他的git GUI软件甚至是git本身都是可以的）
3. 内核源码 x1 （这里会使用[流念](https://github.com/wloot)和[Vantoman](https://github.com/vantoman)的内核做示例（感谢这两位大佬））
4. git操作手册，用于查询一些基本命令的详细用法。（这里推荐[廖雪峰](https://www.liaoxuefeng.com/wiki/896043488029600)大佬的。）
5. 配置git（照着上面的操作手册的安装那一章节配置就好）
6. 配置GitKraken（官网有详细的配置方法）
7. clone源码（可以使用git clone命令，也可以使用GitKraken软件进行clone，这里使用的是GitKraken）
![克隆中](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/cloning.png)
![克隆完毕](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/cloned.png)

### 合并其他内核的特性到当前内核
1. 我们首先要分析要合并的特性，在本例中，我们在流念的p-miui-eas分支上进行操作（即合并其他内核的某个特性到p-miui-eas分支上，不明白什么是分支的请默默去看上面的git操作手册。），合并vantoman的内核的pwrutilx调速器。
2. 找到你要合并的特性在哪些commit上：
  - 先在vamtonkernel中搜索带pwrutilx的commit
![搜索commit](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/search_commit.png)
  - 我们找到了一个添加pwrutilx调速器的commit和一个开启pwrutilx调速器的commit。
3. 按顺序合并commit：
  - 正常人都知道，一个东西要先添加在开启，所以我们认定这两个commit的关系为[开启调速器]依赖于[添加调速器]，所以我们先合并添加调速器的commit。
  - 首先我们先添加一个remote，指向vamtomkernel的地址（只要添加一次就好，后面不用再添加）
![添加remote](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/add_remote.png)
![添加remote中](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/adding_remote.png)
  使用git的话可以使用以下命令：
  `git remote add [随便给个名字] [目标内核地址]`这条命令是添加远程仓库
  `git fetch [你之前随便给的名字]`这条命令是更新本地的远程仓库的数据
  - 添加完以后就像这样：
![添加完以后](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/added_remote.png)
  - 我们开始合并这两个commit，由于GitKraken显示问题，过于古老的commit可能会找不到，所以这里添加commit我们要使用一下git
  使用`git cherry-pick [对应commit的特征码]`来添加commit即可，第二个commit如法炮制。
  - 如果一切正常，应该不会提示任何错误，那么第四步就可以不看了。
4. 出现冲突
  - 如果出现了类似以下错误的错误，那么就说明出现了冲突，打开GitKraken，软件会自动侦测到冲突
`error: 不能应用 eda7bb4f3bea... Add PWRUTILX EAS Sched
提示：冲突解决完毕后，用 'git add <路径>' 或 'git rm <路径>'
提示：对修正后的文件做标记，然后用 'git commit' 提交`
![侦测到冲突](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/found_conflict.png)
  - 点击View Conflict进入处理冲突界面
![冲突界面](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/conflict.png)
  - 不要点击mark resolved或mark all resolved，这是将冲突文件标记为已解决的意思。我们直接点击文件，这里只有一个冲突文件，所以开始解决这个冲突。
![进入冲突解决界面](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/makefile_conflict.png)
  - 解释一下这个界面，左边这个窗口是当前在p-miui-eas分支上的文件现状，右边是这个commit文件中的内容，下方是输出文件，点击高亮代码行的左边的加号将你所需要的代码添加到输出中。
![冲突已解决](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/resolved.png)
  - 双击save按钮保存，自动回到上级页面，现在没有冲突了。
![冲突解决后](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/no_conflict.png)
  - 杜撰一下你的更改，方便以后查log
![更改](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/commit.png)
  - 点击commit change提交更改
![更改已提交](https://raw.githubusercontent.com/grislux55/Android_Kernel_Magic/master/images/commited.png)
5. 编译内核
6. 测试