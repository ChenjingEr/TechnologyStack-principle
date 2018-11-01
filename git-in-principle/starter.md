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
    3. 
    




