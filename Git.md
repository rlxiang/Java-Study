## Git使用

学习git之前，我们需要先明白一个概念，==版本控制==

### 版本控制

什么是版本控制

版本控制( Revision control )是一种在开发的过程中用于管理我们对文件、目录或工程等内容的修改历史，方便查看更改历史记录，备份以便恢复以前的版本的软件工程技术。

- 实现跨区域多人协同开发
- 追踪和记载一个或者多个文件的历史记录
- 组织和保护你的源代码和文档
- 统计工作量
- 并行开发、提高开发效率
- 跟踪记录整个软件的开发过程
- 减轻开发人员的负担，节省时间，同时降低人为错误
- 简单说就是用于管理多人协同开发项目的技术。

没有进行版本控制或者版本控制本身缺乏正确的流程管理，在软件开发过程中将会引入很多问题，如软件代码的一致性、软件内容的冗余、软件过程的事物性、软件开发过程中的并发性、软件源代码的安全性，以及软件的整合等问题。

无论是工作还是学习，或者是自己做笔记，都经历过这样一个阶段!我们就迫切需要一个版本控制工具！

![image-20210727165756192](.\typora-user-images\image-20210727165756192.png)

多人开发就必须要使用 版本控制，否则代价会比较大！

**常见的版本控制工具**

主流的版本控制器有如下这些：

- **Git**
- **SVN** ( Subversion )
- **cvs** ( concurrent Versions System )
- **vss** ( Micorosoft Visual SourceSafe )
- **TFS** ( Team Foundation Server )
- **visual Studio Online**

版本控制产品非常的多( Perforce、Rational ClearCase、RS ( GNU Revision Control System ) 、 Serena Dimention、SVK,BitKeeper、Monotone、Bazaar、Mercurial、SourceGear Vault )，现在影响力最大且使用最广泛的是Git与SVN

**版本控制分类**

1. **本地版本控制**

   记录文件每次的更新，可以对每个版本做一个快照，或是记录补丁文件，适合个人用，如RCS。

   ![image-20210727170335696](.\typora-user-images\image-20210727170335696.png)

2. **集中版本控制**

   所有的版本数据都保存在服务器上，协同开发者从服务器上同步更新或者上传自己的修改

   ![image-20210727170637929](.\typora-user-images\image-20210727170637929.png)

   所有的版本数据都存在服务器上，用户的本地只有自己以前所同步的版本，如果不连网的话，用户就看不到历史版本，也无法切换版本验证问题，或在不同分支工作。而且，所有数据都保存在单一的服务器上，有很大的风险这个服务器会损坏，这样就会丢失所有的数据，当然可以定期备份。代表产品:SVN、CVS、VSS

3. **分布式版本控制 Git**

   每个人都拥有全部的代码！会产生安全隐患！

   所有版本信息仓库全部同步到本地的每个用户，这样就可以在本地查看所有版本历史，可以离线在本地提交，只需在连网时push到相应的服务器或其他用户那里。由于每个用户那里保存的都是所有的版本数据，只要有一个用户的设备没有问题就可以恢复所有的数据，但这增加了本地存储空间的占用。

   不会因为服务器损坏或者网络问题，造成不能工作的情况！

   ![image-20210727171011030](.\typora-user-images\image-20210727171011030.png)

**Git 与 SVN 的区别**

SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而工作的时候，用的都是自己的电脑，所以首先要从中央服务器得到最新的版本，然后工作，完成工作后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，对网络带宽要求较高。

Git是分布式版本控制系统，没有中央服务器，每个人的电脑就是一个完整的版本库，工作的时候不需要联网了，因为版本都在自己电脑上。协同的方法是这样的︰比如说自己在电脑上改了文件A,其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。Git可以直接看到更新了哪些代码和文件!

==Git是目前世界上最先进的分布式版本控制系统。==



### Git历史

同生活中的许多伟大事物一样，Git诞生于一个极富纷争大举创新的年代。

Linux内核开源项目有着为数众广的参与者。绝大多数的Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上(1991-2002年间)。到2002年，整个项目组开始启用一个专有的分布式版本控制系统BitKeeper来管理和维护代码。

到了2005年，开发BitKeeper的商业公司同Linux 内核开源社区的合作关系结束，他们收回了Linux内核社区免费使用

BitKeeper的权力。这就迫使Linux开源社区(特别是Linux的缔造者Linus Torvalds)基于使用BitKeeper时的经验教训，开发出自己的版本系统。也就是后来的Git!

Git是目前世界上最先进的分布式版本控制系统。

Git是免费、开源的，最初Git是为辅助Linux内核开发的，来替代 BitKeeper !



### Git环境配置

Git 镜像地址 http://npm.taobao.org/mirrors/git-for-windows/

下载对应的版本

无脑下一步即可



卸载的时候，先删掉环境变量，然后在卸载即可



>启动Git

安装成功后在开始菜单中会有Git项，菜单下有3个程序，任意文件夹下右键也可以看到对应的程序！

![image-20210727194427094](.\typora-user-images\image-20210727194427094.png)

**Git GUI**：图形界面的Git，不建议初学者使用，尽量先熟悉常用命令

**Git Bash**：Unix与Linux风格的命令行，使用最多，推荐使用

**Git CMD**：Windows风格的命令行

>基本的Linux命令

- cd : 改变目录
- cd.. : 回退到上一级目录 ，直接cd进入默认目录
- pwd:显示当前所在的目录路径。
- ls(Il):都是列出当前目录中的所有文件，只不过(两个II)列出的内容更为详细。
- touch:新建一个文件如touch index.js就会在当前目录下新建一个index.js文件。
- rm:删除一个文件, rm index.js 就会把index.js文件删除。
- mkdir:新建一个目录,就是新建一个文件夹。
- rm -r:删除一个文件夹, rm -r src 删除src目录
- mv移动文件, mv index.html src ，index.html是我们要移动的文件, src是目标文件夹,当然，这样写,必须保证文件和目标文件夹在同一目录下。
- resetl重新初始化终端/清屏。
- clear清屏。
- history查看命令历史。
- help帮助。
- exit退出。
- #表示注释

>Git  配置

查看配置  **git config -l**

![image-20210727200611641](.\typora-user-images\image-20210727200611641.png)

查看不同级别的配置文件：

```
#查看系统config
git config --system --list
#查看当前用户（global）配置
git config --global --list
```

**Git相关的配置文件：**

1）Git\etc\gitconfig ： Git 安装目录下的 gitconfig --system 系统级

![image-20210727201522067](.\typora-user-images\image-20210727201522067.png)

2）C:\Users\Administrator\ .gitconfig 只适用于当前登录用户的配置 --global 全局(没有设置的时候里面什么都没有，如果没有该文件，就自己建立一个就行)

![image-20210727201949062](.\typora-user-images\image-20210727201949062.png)

这里可以直接编辑配置文件，通过命令设置后会响应到这里。

>==设置用户名与邮箱(用户标识，必须要配置)==

当你安装Git后首先要做的事情是设置你的用户名称和e-mail地址。这是非常重要的，因为每次Git提交都会使用该信息。但是这里的信息是可以自己乱写的。主要作为标识使用。它被永远的嵌入到了你的提交中:

```bash
git config --global user.name "rlxiang" #名称
git config --global user.email 594663327@qq.com #邮箱
git config --list  查看配置信息
```

只需要做一次这个设置，如果你传递了--global选项，因为Git将总是会使用该信息来处理你在系统中所做的一切操作。如果你希望在一个特定的项目中使用不同的名称或e-mail地址，你可以在该项目中运行该命令而不要--global选项。总之--global为全局配置，不加为某个项目的特定配置。

### Git基本理论（核心）

>工作区域

Git本地有三个工作区域:工作目录(Working Directory )、暂存区(Stage/Index)、资源库(Repository或cit Directory)。如果在加上远程的git仓库(Remote Directory)就可以分为四个工作区域。文件在这四个区域之间的转换关系如下︰

![image-20210727202717540](.\typora-user-images\image-20210727202717540.png)

- Workspace :工作区，就是你平时存放项目代码的地方
- Repository :仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
- Remote:远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换

本地的三个区域确切的说应该是git仓库中HEAD指向的版本:

![image-20210727203419784](.\typora-user-images\image-20210727203419784.png)

- Directory:使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。
- WorkSpace:需要通过Git进行版本控制的目录和文件，这些目录和文件组成了工作空间。
- .git:存放Git管理信息的目录，初始化仓库的时候自动创建。
- Index/Stage:暂存区，或者叫待提交更新区，在提交进入repo之前，我们可以把所有的更新放在暂存区。
- Local Repo:本地仓库，一个存放在本地的版本库;HEAD会只是当前的开发分支( branch )。
- Stash:隐藏，是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

>工作流程

git的工作流程一般是这样的：

1. 在工作目录中添加、修改文件；（比如新建了一个UserMapper）
2. 将需要进行版本管理的文件放入暂存区域；(git add .)
3. 将暂存区域的文件提交到git仓库。(git commit)

因此，git管理的文件有三种状态：已修改（modified）,已暂存（staged）,已提交(committed)

![image-20210727203849682](.\typora-user-images\image-20210727203849682.png)

### Git项目搭建

>创建工作目录与常用指令

工作目录（WorkSpace)一般就是你希望Git帮助你管理的文件夹，可以是你项目的目录，也可以是一个
空目录，建议不要有中文。

日常使用只要记住下图6个命令：

![image-20210727203949558](.\typora-user-images\image-20210727203949558.png)

>本地仓库搭建

创建本地仓库的方法有两种：一种是创建全新的仓库，另一种是克隆远程仓库。

1. 创建全新的仓库，需要用GIT管理的项目的根目录(工作目录)执行：

   ```bash
   // 在当前目录新建一个Git仓库
   git init
   ```

2. 执行后可以看到，仅仅在项目目录多出了一个.git目录(版本库)，关于版本等的所有信息都在这个目录里面。

   ![image-20210727204423585](.\typora-user-images\image-20210727204423585.png)

>克隆远程仓库

1. 另一种方式是克隆远程目录，由于是将远程服务器上的仓库完全镜像一份至本地！

   ```bash
   # 克隆一个项目和它的整个代码历史(版本信息)
   $ git clone [url] https://github.com/rlxiang/Java-Study.git
   ```

2. 去 gitee 或者 github 上克隆一个测试！



### GIT文件操作

>文件4中状态

版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在还不想提交的文件，或者要提交的文件没提交上。

- **Untracked**:未跟踪，此文件在文件夹中,但并没有加入到git库,不参与版本控制.通过git add状态变为Staged
- **Unmodify**:文件已经入库,未修改，即版本库中的文件快照内容与文件夹中完全一致.这种类型的文件有两种去处,如果它被修改,则变为Modified .如果使用git rm移出版本库,则成为untracked文件
- **Modified**:文件已修改,仅仅是修改,并没有进行其他的操作.这个文件也有两个去处,通过git add可进入暂存staged状态,使用git checkout则丢弃修改过,返回到unmodify状态,这个 git checkout即从库中取出文件,覆盖当前修改!
- **Staged**:暂存状态.执行 git commit则将修改同步到库中,这时库中的文件和本地文件又变为一致，文件为_unmodify状态.执行git reset HEAD filename取消暂存,文件状态为Modified

>查看文件状态

上面说文件有4种状态，通过如下命令可以查看到文件的状态:

```bash
#查看指定文件状态
git status [filename]

#查看所有文件状态
git status


# git 本地仓库常用命令
 git add .      添加所有文件到暂存区
 git add [filename] 添加指定文件到暂存区
 git commit -m "消息内容" filename 提交暂存区中的内容到本地仓库 -m表示提交信息
 git reset [filename] 将暂存区的文件取消暂存或者是切换到指定版本
 git reset --hard 日志版本(一串序列号)   执行即可回到对应的版本
 git log  查看日志
```

![image-20220503163219517](.\typora-user-images\image-20220503163219517.png)



```bash
# git 远程仓库常用命令
git remote [-v]      查看远程仓库
git remote add [shortname] [url] 添加远程仓库,就是将本地仓库与远程仓库关联起来实施通信。该命令必须要先存在远程仓库才能执行。shortname一般指定为origin

git clone       从远程仓库克隆
git pull [short-name] [branch-name]        从远程仓库拉取
注意：如果当前本地仓库不是从远程仓库中克隆过来的，而是本地创建的仓库，并且仓库中存在文件，此时再从远程拉取文件的时候会报错(fatal: refusing to merge unrelated histories)
解决此问题可以在git pull命令后加入参数 --allow-unrelated-histories

git push [remote-name] [branch-name]       推送到远程仓库
```



### git 分支

分支是Git 使用过程中非常重要的概念。使用分支意味着你可以把你的工作从开发主线上分离开来，以免影响开发主线。

同一个仓库可以有多个分支，各个分支相互独立，互不干扰。

通过git init 命令创建本地仓库时默认会创建一个master分支

```bash
关于分支的相关命令，具体如下：
 git branch			            查看分支
 git branch [name]		        创建分支
 git checkout [name]		    切换分支
 git push [shortName] [name]	推送至远程仓库分支
 git merge [name]		        合并分支
##################################################
git branch	列出所有本地分支
git branch -r	列出所有远程分支
git branch -a	列出所有本地分支和远程分支

git branch [name] :在本地创建分支命令
git checkout [name] ：切换本地分支命令
git push [shortName] [name] ：将本地分支推送至远程仓库分支命令
git merge [name]          ：合并分支命令，合并后只存在于本地，若要在远程仓库中存在还需要push
###############################################
分支合并出现异常的情况： 在不同的分支里修改同一文件后，再进行合并可能会报错，需要先解决冲突，在合并。解决方法如下：

首先将合并的文件中不需要的信息删掉，然后使用 git add 合并的文件名（b1.txt）， 这一操作就是告诉git，冲突已经解决了，
然后再提交 git commit -m "..." b1.txt, 如果这一步报错：fatal：cannot do a partial commit during merge,
则可以在提交时加入 -i 参数，git commit -m "..." b1.txt -i. 然后再重新推送该文件即可(b1.txt)
```



### 标签操作

Git 中的标签，指的是某个分支某个特定时间点的状态。通过标签，可以很方便的切换到标记时的状态。

比较有代表性的是人们会使用这个功能来标记发布结点（v1.0 、v1.2等）。下面是mybatis-plus的标签：

![image-20220503205919851](.\typora-user-images\image-20220503205919851.png)



>git 标签常用命令

```bash
git tag			列出已有的标签
git tag [tagName]			创建标签
git push [shortName] [tagName]	将标签推送至远程仓库
git checkout -b [branch] [tagName]	检出标签, 即可以将当时这个标签的状态下载下来，就能拿到当时标签状态下的代码.检出标签时需要新建一个分支来指向某个标签

```









>忽略文件

有些时候我们不想把某些文件纳入版本控制中，比如数据库文件，临时文件，设计文件等

在主目录下建立".gitignore"(springboot中会自动生成该文件)文件，此文件有如下规则︰

1. 忽略文件中的空行或以井号(# ）开始的行将会被忽略。
2. 可以使用Linux通配符。例如:星号(*)代表任意多个字符，问号( ?)代表一个字符，方括号([abc])代表可选字符范围，大括号( {string1,string2,.…})代表可选的字符串等。
3. 如果名称的最前面有一个感叹号(!），表示例外规则，将不被忽略。
4. 如果名称的最前面是一个路径分隔符(/ )，表示要忽略的文件在此目录下，而子目录中的文件不忽略。
5. 如果名称的最后面是一个路径分隔符(/)，表示要忽略的是此目录下该名称的子目录，而非文件（默认文件或目录都忽略)。

```bash
#为注释
*.txt          #忽略所有.txt结尾的文件,这样的话上传就不会被选中!
!1ib.txt       #但1ib.txt除外
/temp          #仅忽略项目根目录下的TODO文件,不包括其它目录temp
build/         #忽略build/目录下的所有文件
doc /*.txt     #会忽略 doc/notes.txt但不包括doc/server/arch.txt
```

idea中可忽略的文件

```bash
*.class
*.log
*.lock


# Package Files #
*.jar
*.war
*.ear

# idea
.idea/
*.iml

*velocity,log*

### STS ###
.apt_generated
.factorypath
.springBeans

### Intellij IDEA ###
*.iml
*.ipr
*.iws
.idea
.classpath
.project
.settings/
bin/

*.log
tmp/

#rebel
*rebel.xml*
```



### 使用码云

1. 登录码云，完善信息

2. 设置本机绑定SSH公钥，实现免密码登录！(免密码登录，这一步挺重要，码云是远程仓库，我们是平时工作在本地仓库！不然的后就会每次提交都要输入密码)

   ```bash
   #进入c:\usersAdministrator.ssh目录
   #生成公钥
   ssh-keygen
   ```

   ![image-20210727221652337](.\typora-user-images\image-20210727221652337.png)

3. 将公钥信息public key添加到码云账户中即可!

   ![image-20210727222052461](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20210727222052461.png)

4. 使用码云创建一个自己的仓库!

   ![image-20210727222620096](.\typora-user-images\image-20210727222620096.png)

   ​	然后把这个仓库给克隆下来就能用了



### IDEA中集成Git

1. 新建项目，绑定Git

   - 将我们远程的git文件目录拷贝到项目中即可！
   - ![image-20210728104053472](.\typora-user-images\image-20210728104053472.png)
   - 注意观察idea中的变化
   - ![image-20210728104341892](.\typora-user-images\image-20210728104341892.png)

2. 修改文件，使用IDEA操作Git

   - 添加到暂存区
   - commit 提交
   - push到远程仓库

3. 提交测试

   ![image-20210728110617834](.\typora-user-images\image-20210728110617834.png)

这些都是单人操作



### 说明：GIT分支

分支在GIT中相对较难，分支就是科幻电影里面的平行宇宙，如果两个平行宇宙互不干扰，那对现在的你也没啥影响。不过，在某个时间点，两个平行宇宙合并了，我们就需要处理一些问题了!

![image-20210728111102573](.\typora-user-images\image-20210728111102573.png)

![image-20210728110730052](.\typora-user-images\image-20210728110730052.png)

git 分支中常用的命令

```bash
# 列出所有本地分支
git branch

# 列出所有远程分支
git branch -r

# 新建一个分支，但依然停留在当前分支
git branch [branch-name]

# 新建一个分支，并切换到该分支
git checkout -b [branch]

# 合并指定分支到当前分支
git merge [branch]

# 删除分支
git branch -d [branch-name]

# 删除远程分支
git push orgin --delete [branch-name]
git branch -dr [remote/branch]
```

多个分支如果并行执行，就会导致我们代码不冲突，也就是同时存在多个版本

如果了冲突了就需要协商即可!|

如果同一个文件在合并分支时都被修改了则会引起冲突∶解决的办法是我们可以修改冲突文件后重新提交Ⅰ选择要保留他的代码还是你的代码!

master主分支应该非常稳定，用来发布新版本，一般情况下不允许在上面工作，工作一般情况下在新建的dev分支上工作，工作完后，比如上要发布，或者说dev分支代码稳定后可以合并到主分支master上来。

![image-20210728112534297](.\typora-user-images\image-20210728112534297.png)



### 在idea中操作git分支

- 查看分支

- 创建分支

![image-20220505213816171](.\typora-user-images\image-20220505213816171.png)

- 切换分支
- 将分支推送到远程仓库
- 合并分支



