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
#### 2. 再 push 上去
这里要强行 push ，-f
```
$ git push origin xxxxx -f
```


