# 1. git的工作流程

![截图](f2a5bc56e00176c262b252c12dc7d4d8.png)

<br/>

1. clone（克隆）: 从远程仓库中克隆代码到本地仓库
2. checkout （检出）:从本地仓库中检出一个仓库分支然后进行修订
3. add（添加）: 在提交前先将代码提交到暂存区
4. commit（提交）: 提交到本地仓库。本地仓库中保存修改的各个历史版本
5. fetch (抓取) ： 从远程库，抓取到本地仓库，不进行任何的合并动作，一般操作比较少。
6. pull (拉取) ： 从远程库拉到本地库，自动进行合并(merge)，然后放到到工作区，相当于 fetch+merge
7. push（推送） : 修改完成后，需要和团队成员共享代码时，将代码推送到远程仓库

# 2.git常用命令

## 2.1 篇日志用户信息

### 基本配置

git config --global user.name "姓名"    \#配置姓名

git config --global user.email "邮件地址" \#配置邮件地址

### 查看配置信息

 git config --global user.name  \#查看姓名

git config --global user.email   \#查看邮件

## 解决乱码问题

gitbash执行代码

git config --global core.quotepath false

## 获取本地仓库

git init

## 文件状态变化

![截图](b58d2b0e66a271282623c43a42fd6b33.png)

### 将指定文件加入暂存区

git add 文件名

### 将所有文件加入暂存区  

git add .  

### 将暂存区文件加入本地仓库备注为注释内容

git commit -m "注释内容"

### 查看修改的状态  

git status  

### 查看提交记录

git log \[option]  
        options：
            --all 显示所有分支
            --pretty=oneline 将提交信息显示为一行
            --abbrev-commit 使得输出的commitId更简短
            --graph 以图的形式显示

### 版本回退

git reset --hard commitID

```
     commitID 可以使用git log 指令查看的数字码
```

如何查看已经删除的记录？
git reflog
这个指令可以看到已经删除的提交记录

### 添加文件至忽略列表

一般我们总会有些文件无需纳入Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动
生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以在工作目录
中创建一个名为 .gitignore 的文件（文件名称固定），列出要忽略的文件模式。下面是一个示例：

```
# 忽视.a格式文件
*.a
# 跟踪.a格式文件，即使你忽略.a文件
!lib.a
# 只忽略当前目录的TODO文件
/TODO
# 忽略build文件夹下的所有文件
build/
# 忽略doc文件夹下的所有txt格式文件，但不包括doc中的文件夹中的txt文件
doc/*.txt
# 忽略doc文件下的所有txt格式文件，即使在子文件夹中
doc/**/*.pdf
```

## 分支

### 查看本地分支

git branch

### 创建本地分支

git branch 分支名

### *切换分支

git checkout 分支名

### 创建并且切换到分支

git checkout -b 分支名

### *合并分支

git merge 分支名

### 删除分支

git branch -d b1 删除分支，需要做各种检查

git branch -D b1 删除分支，不做检查

### 解决冲突

1. 处理文件中冲突的地方
2. 将解决完冲突的文件加入暂存区(add)
3. 提交到仓库(commit)

### 分支使用原则

#### master（生产）分支

线上分支，主分支，中小规模项目作为线上运行的应用对应的分支；

#### develop（开发）分支

是从master创建的分支，一般作为开发部门的主要开发分支，如果没有其他并行开发不同期上线要求，都可以在此版本进行开发，阶段开发完成后，需要是合并到master分支,准备上线。

#### feature分支

从develop创建的分支，一般是同期并行开发，但不同期上线时创建的分支，分支上的研发任务完成后合并到develop分支。

#### hotfix分支

从master派生的分支，一般作为线上bug修复使用，修复完成后需要合并到master、test、develop分支

![截图](71cb91478e92fabc704e507a688db7c5.png)

### git远程仓库

## 配置SSH公钥

### 生成SSH公钥

ssh-keygen -t rsa

一直回车

新公钥会覆盖旧公钥

### gitee设置账户

获取公钥

cat ~/.ssh/id_rsa.pub

验证是否成功

ssh -t git@gitee.com

## 操作远程仓库

### 添加远程仓库

此操作是先初始化本地库，然后与已创建的远程库进行对接。

git remote add <远端名称> <仓库路径>
远端名称，默认是origin，取决于远端服务器设置
仓库路径，从远端服务器获取此URL
例如: git remote add origin git@gitee.com:czbk_zhang_meng/git_test.git

### 查看远程仓库

git remote

### 推送到远程仓库

git push \[-f\] \[--set-upstream] \[远端名称] \[ 本地分支名:远端分支名  \]
如果远程分支名和本地分支名称相同，则可以只写本地分支
git push origin master
-f 表示强制覆盖
--set-upstream 推送到远端的同时并且建立起和远端分支的关联关系。
git push --set-upstream origin master
如果当前分支已经和远端分支关联，则可以省略分支名和远端名。
git push 将master分支推送到已关联的远端分支。

### 本地分支与远程分支的关联关系

查看关联关系我们可以使用 git branch -vv 命令

### 从远程仓库克隆

如果已经有一个远端仓库，我们可以直接clone到本地。
命令: git clone <仓库路径> \[本地目录]
本地目录可以省略，会自动生成一个目录

### 从远程仓库中抓取和拉取

远程分支和本地的分支一样，我们可以进行merge操作，只是需要先把远端仓库里的更新都下载到本地，再进行操作。

**抓取 命令：git fetch \[remote name] \[branch name]**

抓取指令就是将仓库里的更新都抓取到本地，不会进行合并
如果不指定远端名称和分支名，则抓取所有分支。

**拉取 命令：git pull \[remote name] \[branch name]**

拉取指令就是将远端仓库的修改拉到本地并自动进行合并，等同于fetch+merge
如果不指定远端名称和分支名，则抓取所有并更新当前分支。

### 解决远端合并冲突

在一段时间，A、B用户修改了同一个文件，且修改了同一行位置的代码，此时会发生合并冲突。A用户在本地修改代码后优先推送到远程仓库，此时B用户在本地修订代码，提交到本地仓库后，也需要推送到远程仓库，此时B用户晚于A用户，故需要先拉取远程仓库的提交，经过合并后才能推送到远端分支,如下图所示。

**远程分支也是分支，所以合并时冲突的解决方式也和解决本地分支冲突相同相同**

# PS

1. 切换分支前先提交本地的修改
2. 代码及时提交，提交过了就不会丢