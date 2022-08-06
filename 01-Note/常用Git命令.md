## [Git的4个阶段的撤销更改](https://segmentfault.com/a/1190000011969554)

1、已修改，未暂存(未add)			git checkout .

2、已暂存，未提交(未commit)		git reset --hard

3、已提交，未推送(未push)		git reset --hard HEAD~1

4、已推送	

5、删除新增加文件（未add）

6、强行删除本地的未tracked文件		git clean -fd -n



git add 后撤销： 

​	撤销所有add文件 git reset HEAD .

​    撤销单个add文件 git reset HEAD -filename



git commit 后撤销：

​	只回退commit的信息，保留修改代码：git reset --soft head

​	彻底回退到上次commit版本，不保留修改代码：git reset --hard head^

​	说明：

​	HEAD ：当前版本

​	HEAD^ ：上一个版本

​	--hard 参数会抛弃当前工作区的修改

​	--soft 参数的话会回退到之前的版本，但是保留当前工作区的修改，可以重新提交



撤销所有本地改动代码： 

​	git checkout .

本地代码回退到与git远程仓库保持一致 

​	git reset --hard 远程分支名

git push撤销： 

​	回滚此次push到服务器的代码：

​	git log查看commit的信息

​	git revert 以前commit的id

​	git push 此时本地回滚的代码到服务器就可以了



git merge 撤销： 

​	$ git checkout 【行merge操作时所在的分支】

​	$ git reset --hard 【merge前的版本号】





## 从某一个**commit**开始创建本地分支 

git checkout -b 本地新branchName commitId

git checkout -b testBr c485217 //分支testBr是基于c485217创建，创建的同时并切换到此分支。



## 添加过.gitignore

　　git rm -r --cached .

　　git add .

　　git commit -m 'update .gitignore'

　　git push