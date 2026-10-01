# Git笔记
## Git安装
**一些命令**  
* ls/ll 查看当前目录
* cat 查看文件内容
* touch 创建文件
* vi   vi编辑器

### Git下载与安装
git下载地址 [点击](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
，一路点击确认到最后  
**基本配置**
1. 打开git bash
2. 设置用户信息
```git
git config --global user.name "yourname"
git config --global user.email "yourEmail"
```
查看配置信息
```git
git config --global user.name
git config --global user.email
```

### 为常用指令配置别名(可选)
1. 打开用户目录，创建.bashrc文件
```markdown
touch ~/.bashrc
```
2. 在.bashrc文件中输入如下内容
```markdown
#日志别名
alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'
#输出当前目录下所有文件及基本信息
alias ll='ls -al'
```
3. 打开gitBash，执行`source ~/.bashrc`，使bashrc文件起作用

**解决GitBash乱码问题**
1. 打开gitBash执行``git config core.quotepath false`
2. ${git_home}/etc/bash.bashrc 文件最后下面加两行
```markdown
export LANG="zh_CN.UTF-8"
export LC_ALL="zh_CN.UTF-8"
```

## git操作
### 获取本地仓库
1. 找到想要作为仓库的文件夹并进入该文件夹
2. 在目录页打开GitBash窗口
3. 执行命令`git init`
4. 若成功就可以看见一个.git隐藏文件

### 基础操作指令
**查看修改状态**`git status`  
**提交修改流程**：
1. git add 文件名|.
2. git commit -m "注释"  

**查看日志**：`git log`或者`git-log`（前面提过的别名）  
**版本回退**：`git reset --hard commitID`
**查看已经删除的记录**：`git reflog`  

### git分支操作
* 查看本地分支：`git branch`
* 查看分支及其关联的远程分支：`git branch -vv`
* 创建本地分支：`git branch 分支名`
* 切换分支（checkout）：`git checkout 分支名`
* 创建并切换：`git checkout -b 分支名`
* 合并分支（merge）：`git merge 分支名`   通常合并到master分支上
* 删除分支：  
不能删除当前分支，只能删除其他分支
    * 普通删除`git branch -d 分支名`
    * 强制删除`git branch -D 分支名`

### 解决冲突
打开冲突的文件，具体怎么处理，删除哪一个冲突操作或者作新修改，由操作者决定

## Git远程仓库
> 常用的远程仓库由github或者码云（gitee），企业gitlab
### 配置SSH公钥
1. 生成SSH公钥：在命令行输入`ssh-keygen -t rsa`，不断回车，如果公钥已经存在则自动覆盖。
2. 远程仓库添加账户公钥
    * 获取公钥：`cat ~/.ssh/id_rsa.pub`
    * 复制其中内容到远程仓库的公钥
    * 验证是否成功：`ssh -T git@github.com` 或者 `ssh -T git@gitee.com`
### 操作远程仓库
1. 添加远程仓库：
打开当前仓库bash输入`git remote add <远端名称(origin)> <仓库名称>  
```markdown
git remote add origin ssh仓库地址
```
2. 查看远程仓库：`git remote`，`git remote -v`
3. 推送到远程仓库
命令：`git push [-f] [--set-upstream] [远端名称[本地分支名][:远端分支名]]`
> * 如果远程分支名和本地分支名相同，则只写本地分支：`git push origin master`
> * --set-upstream 推送到远端的同时建立起和远端分支的关联关系。`git push --set-upstream origin master`
> * 如果当前分支已经和远端分支关联，则可以直接省略分支名和远端名。`git push`

4. 从远端克隆：`git clone <仓库地址> [本地目录]`
5. 从远程仓库抓取和拉取：
* 抓取指令：`git fetch [remote name] [branch name]`
    - 抓取指令就是把更新都抓取到本地，不会进行合并分支。
    - 如果不指定远端名称和分支名，则抓取所有分支。
* 拉取指令：`git pull [remote name] [branch name]`
    - 拉取指令就是将抓取并合并，等同于fetch+merge
    - 如果不指定远端名称和分支名，则拉取所有更新并合并当前分支。

### 工作流程
```
git remote add origin  <仓库名称>
git clone <仓库地址> [本地目录]
git-log

# 一系列更新操作后

git add .
git commit -m '注释'
# 第一次push
git push --set-upstream origin master:master
# 后续push
git push

# 之后每次push前先pull
git pull

```

