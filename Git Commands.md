# git 使用指南

## 配置

```bash
git config --global user.name Trace_Shek
git config --global user.email joesixpack@foxmail.com
```

## 克隆

```bash
git clone giturl.git
```

## 删除

```bash
git clean -n 显示那些文件会被删除
git clean -f 删除当前目录下没有被跟踪过的文件，但不会删除 .gitignore 指定的文件。
git clean -df 删除当前目录下没有被跟踪过的文件和文件夹
```

## 暂存

```bash
git add . 提交所有修改和新增的文件
git add -u 只提交修改文件而不提交新文件
git checkout . 放弃没有提交的所有修改
git checkout file.f 放弃指定文件的修改
git ls-files -s 查看暂存区文件列表
git cat-file -p 查看暂存区文件内容 
```

## 恢复

```bash
git reset --hard 清空工作区和暂存区的改动
git reset --hard HEAD^^^ 恢复前三个版本
git reset --soft 保留工作区内容，把文件差异放进暂存区
git reset --hard 恢复到指定提交版本（git log 查看版本号）
```

## 分支

```bash
git branch dev 创建分支
git branch 查看分支
git checkout dev 切换分支
git checkout -b dev 创建并切换分支
git branch -d dev 删除分支
git branch -D dev 删除没有合并的分支
git push origin :dev 删除远程分支

git checkout master 以下需切换至主分支
git merge dev 合并分支
git branch --no-merged 查看未合并的分支
git branch --merged 查看未合并的分支
```

## 日志

```bash
git log 查看日志
git log -p -3 查看指定次数提交日志并显示文件差异
git log --name-only 显示已修改的文件清单
git log --name-status 显示变更状态
git log --oneline 显示简写版本号
```

## 别名

### 配置文件定义

修改配置文件 ~/.gitconfig 并添加以下命令别名配置段

    [alias]
      a = add .
      c = commit
      s = status
      l = log
      b = branch

### 系统配置定义

修改 ~/.bashrc 或 ~/.bash_profile 文件

    alias gs="git status"
    alias gc="git commit -m "
    alias gl="git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit  "
    alias gb="git branch"
    alias ga="git add ."
    alias go="git checkout"

## 忽略

.gitignore 文件用于定义忽略提交的文件

- 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。
- 匹配模式最后跟反斜杠（/）说明要忽略的是目录。
- 可以使用标准的 glob 模式匹配。

## 存储

```bash
git stash 储藏工作
git stash list 查看储藏列表
git stash apply 应用最近储藏
git stash apply stash@{2} 应用指定储藏
git stash drop stash@{0} 删除储藏
git stash pop 应用并删除储藏
```

## 标记

```bash
git tag v1.0.1 添加标签
git tag 列出标签
git push --tags 推送标签
git tag -d v1.0.1 删除标签
got push origin :v1.0.1 删除远程标签
```

## 打包

```bash
git archive master --prefix='out/' --format=zip > files.zip
```

## 拉取

```bash
git pull origin dev:dev 拉取 origin 主机的 dev 分支与本地的 master 分支合并
git pull origin dev 拉取 origin 主机的 dev 分支与当前分支合并
git pull 拉取本地同名远程分支
```

## 推送

```bash
git push origin 将当前分支推送至 origin 主机的对应分支（如果当前只有一个追踪分支，可省略主机名）
git push -u origin master 使用 -u 选项指定默认主机
git push origin --delete dev 删除远程 dev 分支
git push --set-upstream origin dev 本地 dev 分支关联远程分支并推送
```

# 工作流

## 基础流程

```bash
git clean -n  查看没有被管理的文件
git add . 将所有文件提交至暂存区
git checkout file.f 恢复某个误删除的文件
git commit -m 'first commit'  创建一个新的提交，并使用 -m 选项添加描述
git push 推送至远程服务器
```

## 分支流程

```bash
git branch dev  新建开发分支
git checkout dev  切换到新分支开始开发
git commit -m 'develop' 提交
git checkout master 
git merge dev 合并分支
git push  推送代码
```

## 关联流程

```bash
git init
git add .
git commit -m '初始提交'
git remote add origin url 添加远程仓库
git remote -v 查看远程库
git push -u origin master 推送至远程仓库
git remote rm origin  删除与远程仓库的关联
```
