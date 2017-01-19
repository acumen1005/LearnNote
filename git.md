# git 笔记

## reabse

#### 1. 仓库和上游仓库版本保持一致
```
$ git branch -a
* master
  xxxxx  
```
```
$ git pull upstream master
```
```
$ git checkout xxxxx
Switched to branch 'xxxxx'
``` 
#### 2. 进行 rebase 操作
```
$ git rebase master
``` 
这一步可能会产生冲突，解决冲突， 重新讲冲突的文件加入提交的队列

```
$ git rebase --continue
```
#### 3. 再 push 上去
这里要强行 push ，-f
```
$ git push origin xxxxx -f
```

## 关于 pull/merge request 
Q：团队中缺少不了 code review ，在 code review 中需要修改的在 master 这个分支中其实不需要了解，只需要功能上的 commit message 就好了，其实就需要来 rebase -i 的一个操作

code review 结束之后

```
$ git rebase -i master
```
此时会出现一个编辑 message 的窗口，包含一些提交 message 信息，根据下面 command 的提示做相应的操作。值得注意的是，这里的合并 commit 只能是下面向上面合并。如何合并成功之后，还需要编辑一个 message 作为合并之后的 commit 一个 message。

