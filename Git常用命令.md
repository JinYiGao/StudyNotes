# Git常用命令

```shell
git init  #初始化 

git add .  #添加所有 

git commit -m "first commit"   #提交 并添加注释

git remote add origin https://github.com/JinYiGao/Geological-hazard-weapon  #远程连接到仓库

git push -u origin master  #push到仓库

git pull 
#  若要提交到分支 则 
#  git push -u origin 分支名

git branch #查看当前分支
git branch 分支名   #创建新分支
git checkout -b 分支名    #（你自己创建的分支名 若无 则创建新分支）
git checkout 分支名   #切换分支

git push origin --delete master  #删除Github项目上的master分支

git rm -r --cached "文件路径"   #不删除物理文件，仅将该文件从缓存中删除；

git rm -r --f "文件路径"   #不仅将该文件从缓存中删除，还会将物理文件删除

```

