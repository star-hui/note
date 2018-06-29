# 1. 简单介绍

## 1.1. git起源

       在1991年linus创建了Linux从此linux成为服务器领域的佼佼者，大部分web服务器、邮件、数据库各种服务器端程序都安装在了linux上面运行，主要是因为它运行的快速、高效、利用率高，这样一个优秀的系统并不是一个人在维护，来自民间的众多高手一起在维护这linux发展，那么这么多分布式世界各地的人如何共同维护如此多的Linux代码呢？
       这就需要一个分布式代码管理工具，linus使用过BitKeeper来管理代码但是它是收费的，让很多人用着不爽，后来linus本人就自己开发写了一个工具来管理，这就是git的第一个版本。
       后来随着时间推移越来越多的开源软件通过Git来管理，为了把世界各地的开源项目管理起来，GitHub网站随后上线了，很多流行的项目加入的此网站上面。例如我们经常使用的jquery等。

## 1.2. 集中式vs分布式

      分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，也就是有没有联网都可以正常工作，而SVN在没有联网的时候是拒绝干活的！当有网络的时候，再把本地提交推送一下就完成了同步，真是太方便了！


      集中式版本控制系统像CVS、SVN等，一般是将代码部署到一台服务器上面，每个开发者在每天开发之前需要从服务器上面checkout下来最新的代码，本地修改完后要 commit，要求本地电脑与服务器连着才可以提交如果网络带宽慢则会很卡，记得曾经在公司提交代码，几十k的代码每次提交都会卡住，很影响开发速度和开发质量。

      相比之下，git本地就有仓库，每个开发者都有完整的代码，可以在上面进行各种开发，没有网络的延迟。提高开发效率。每个人修改代码之后会将修改的代码互相推送给每个人，通常为了提高互相推送的传输效率往往搭建一台git服务器来进行代码的推送和拉回，可以提高开发效率。

## 1.3. 各种git的介绍

1. git是一个本地工具，用来关联远程的git服务器，同时本地也有时光穿梭的功能
2. gitlab，github，码云等都是云服务，是本地的git的一个镜像。在不同电脑上就可以拉取，同步。

## 1.4. git的安装

一般大神都是用git的命令行工具，同时也有很多的图形化工具:souretree等

1. windows 下的安装
> 从官网上 ：https://git-scm.com/ 上下载，傻瓜式安装就好了

2. linux 下安装

> yum install git

# 2. 本地git的使用

## 2.1. 初始化本地仓库

``` python
# 使用gitbash创建一个目录，并进入到该目录种
mkdir learngit
cd learngit
# 初始化仓库
git init
# 在该文件下多了一个.git的隐藏目录
$ ll -a
total 24
drwxr-xr-x 1 LH 197121 0 6月  28 14:22 ./
drwxr-xr-x 1 LH 197121 0 6月  28 14:20 ../
drwxr-xr-x 1 LH 197121 0 6月  28 14:22 .git/
```

## 2.2. 配置识别账户与密码

这个账户密码并不是登入的账号密码，只是作为一个识别码，区分谁提交的。  
每次commit的时候，都会带上这个信息

``` python
git config --global --list  # 查看是否配置了
git config --global user.name hui
git config --global user.email 232344@qq.com
```

## 2.3. git的操作

使用下面的命令就可以完成一次版本的提交。

``` python
git add filename         # add dir 添加该文件下所有文件; add file1 file2 ; add . 添加所有所有的文件。
git commit -m '注释消息'  # -m是注释的意思。注意的是 每次版本提交都要详细标明每次变化
```

## 2.4. 三区:工作区、缓存区、版本区

1. 工作区就是我们工作的区域--我们编写代码的地方
2. 暂存区，index(stage) -- add的地方
3. 版本区   -- commit的地方

## 2.5. 查看提交的版本信息

``` python
git log
git log --pretty=oneline  # 一行显示，还有其他参数自己研究
```

## 2.6. 版本回退-git reset

``` python
回退到上一个版本  git reset --hard HEAD^
回退到上上个版本  git reset --hard HEAD^^
回到到上100个版本 git reset --hard HEAD~100
回退到指定的版本  git reset --hard 具体的版本号(使用git log查看)
```

但是我们回到上一个版本之后，使用git log 看不到之后的版本了，这个很蛋疼:

1. 使用命令行上面git log可以看到之后的版本，可以到那个版本去。
2. 使用git reflog 可以看到每一次操作的版本号(一般都使用这个)

## 2.7. 撤销-git checkout

> git checkout -- filename

分为两种情况:

> 1. 如果文件自修改后还没有add到缓存区，现在撤销的话，工作区就会和版本库一直
> 2. 如果文件已经添加到缓存区，又做了修改。现在撤销的话，就会回到缓存去的状态

总之，就是让文件回到最后一次git add 或 git commit的状态

# 3. 远端仓库

## 3.1. github设置

1. 再创建账号密码之后，创建新仓库
2. public是公有的，大家都能看见。private是私有的,但是需要money
3. init选项，最好不要选择，不然会造成 本地的master 与 远程的master不一致(本地没readme，远端有readme)，推送的时候会发生错误。可以先pull下来，让本地与云端都有readme。这样可以解决
4. .gitignore是忽略某些文件:一些大文件，我们不需要修改，但又比较大。再push的时候，就可以忽略。增加速度
license是一个公开授权，没啥用


## 3.2. 两种连接方式

1. 使用https的连接方式-适合单人

需要账号密码认证- 适合单人的项目

``` python
git remote add origin https://github.com/star-hui/gittest.git
# origin是仓库的名字，可以随意定义但是下面必须一直
git push -u origin master
# 第一次推送的时候，需要把本地的mster分支与远程的master分支合并。所以加上-u参数
git push origin msster
# 之后每次连接不需要-u参数
```

2. 使用ssh连接-适合团队

直接使用公钥，团队成员每个成员的公钥都加入进来，就可以一起使用了。

``` python
# 第一步：在你本地先生成rsa的密钥
ssh-keygen.exe -t rsa -C '123@qq.com'   # 邮箱是团队每个成员自己的邮箱，只作为标识符

# 第二步: 在上一步生成的rsa文件夹种，把.pub结尾的文件加入到github网站的ssh-key中

# 第三步: 本地进行关联操作
 git remote add louhui git@github.com:star-hui/sshtest.git
   # louhui只是仓库名，:后面的为项目地址，前面邮箱都使用Git官方邮箱。

# 第四部: 推送
# 第一次推送,需要合并本地与远端的master分支
git push -u louhui master
# 以后推送
git push -u louhui master
```

> 在上面其实我们看出，本地的git可以添加多个远程仓库，使用仓库名来区分就可以了。如下:

``` python
git remote add louhui | origin
git push louhui | origin master # 使用不同的仓库名，进行推送
```

## 3.3. 克隆云端的项目

本地不需要新初始化一个仓库，克隆下来就是一个仓库了：  
有两种办法，有些网络场景被屏蔽的一些端口的时候，可以从这两种来选择。

1. 第一种直接使用远端的仓库https地址:进入到仓库，直接复制下来
> git clone https://github.com/star-hui/gittest

2. 第二种使用ssh来连接克隆
> git clone git@github.com:star-hui/sshtest.git

## 3.4. 分支管理

      待完善