```bash
# 恢复为某个 commit 的状态
git reset [commit id] --hard

# git pull 拉取远程内容强制覆盖本地文件
git fetch --all
git reset --hard origin/master
git pull

# 合并 commits
git rebase -i [commit id]

# 远端 repo 相关
git remote show origin     # 显示远程仓库信息
git remote get-url origin  # 显示仓库的 url
git remote show origin     # 可以查看remote地址，远程分支，还有本地分支与之相对应关系等信息

# 远端 branch 改名
git push origin --delete <branch1> # 删除远端 branch1
git branch -m <branch1> <branch2>  # 本地分支改名
git push origin <branch2>

# 删除本地分支
git branch -d <branch name>

# 添加 third_party
git submodule add [repo URL]
```

