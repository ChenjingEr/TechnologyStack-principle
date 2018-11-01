1. setup && config
set up
-> windows
-> linux 
-> mac os

config 
    git config --global user.name ""
    git config --global user.email ""

2. init repository
    git init -> git reply -> empty Git repository .git/
        -> tips: git跟踪文件改动。但是不能够跟踪二进制改动，图片，视频，word文档等
    git add filename -> add file to repository -> add file to stage
    git commit -m "commit info" -> 提交到仓库 -> commit to branch

3. git return back
    update readme.txt (git status) -> modifed ： readme.txt
    git diff -> 查看修改了什么    

4. git log
    git log / git log --pretty=online -> git log format
        ->git 版本 -> HEAD当前版本，HEAD^上一个版本，HEAD^^上上一个版本，HEAD~100前100个
    版本
    git reset --hard HEAD^ -> 回退到上一个版本
    reset回退之后想要返回 -> git reset --hard commit_id(被回退版本的commit id)
    git reflog 查看命令历史

5. git stage
    git add file -> add to stage
    git commit -> commit to branch 
    git diff HEAD -- filename -> 比较stage和branch上filename中的不同

6. 撤销修改
    git checkout -- filename -> 撤销掉当前的修改。git add 之后modify的撤销以及git commit之后modify的撤销
    git reset HEAD filename -> 撤销已经add stage的内容重新返回到workspace

7. remove file
    git rm filename -> remove filename

8. remote resposity
    1. ssh-keygen -r rsa -C "mail address" --> create ssh key --> .ssh id_rsa,id_rsa.pub
    2. github acount setting setting SSH Keys
    3. git remote add origin git_address --> 与远程库关联
    4. git push -u origin master --> push到远程库origin的master分支，u推送+关联
        --> git push origin master --> 推送到master

    ---

    git clone remote_address

9 git branch
    1. git checkout -b dev --> 新建一个dev分支且切换
    2. git branch --> 查看当前分支
    3. git mearge dev --> 合并dev到当前分支 --> fast-forward方式(master指针移动)
    4. git branch -d dev --> 删除dev分支
    
10. git stash
    1. git stash --> saved working directory
    2. git stash list --> 暂存列表
    3. git stash apply --> 恢复列表 --> 恢复后stash并没有删除 --> git stash drop删除 /
    4. git stash pop 恢复的同时删除
    5. 



    




