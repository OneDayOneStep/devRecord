### Git 在非 dev 分支开发

---

- [x] git

+ 之前在切换到 production 分支 merge 完 develop 分支的代码后就忘了切回来，导致后面直接在 production 分支敲了好多砖，准备提交的时候才发现分支错了 😂

```Terminal
 # 添加到版本控制
git add .

 # 把改动的文件提交到储存区
git stash

# 带注释
# git stash save "生产分支敲的代码搬回开发分支"

# 查看储存区
git stash list

# 查看最近一条储存内容
git stash show

# 指定查看储存内容
# git stash show stash@{0}

# 切换到开发分支
git checkout develop

# 把储存区的文件（默认第一个）拉出来并删除
git stash pop

# 把储存区的文件（默认第一个）拉出来但不删除
# git stash apply

# 提交
git commit -m "commit some thing"

# 推送
git push

# 删除储存区 $num 的内容
# git stash drop stash@{$num}

# 清空储存区
# git stash clear
```

+ 如果 git stash show stash@{0} 报出：

<div style="color: red;padding-left: 30px;">
    Too many revisions specified: 'stash@' 'MQA=' 'xml' 'text'
</div>

​	可能是因为当前的终端(Terminal)语法有差异的问题;

​	给 stash@{0} 加上引号即可：git stash show "stash@{0}"



<div align=right>✒️Author: Lee</div>
<div align=right>⏰ RecordTime: 2021-08-27</div>

