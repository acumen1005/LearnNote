# git 笔记

## reabse

#### 1. 仓库和上游仓库版本保持一致
```
$ git branch -a
* master
  photoGallery  
```
```
$ git pull upstream master
```
```
$ git checkout photoGallery
Switched to branch 'photoGallery'
``` 
```
$ git rebase master
``` 
这一步可能会产生冲突，解决冲突， 重新讲冲突的文件加入提交的队列

```
$ git rebase --continue
```
