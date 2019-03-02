<div id="top"></div>
<script src="https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js"></script>
###git全局配置的设置、分支的使用

####设置name和email：
```
git config --global user.name "your name"
git config --global user.email "abc@sina.com"
```
####查看全局配置：
```
git config --list
git config --global --list//或者这个
```
> <span style="background:yellow">注意：git config --list 完后需要按 q 键才能返回</span>

----
####只查看某一项配置：
```
git config user.name
```
```
git config user.email
```
> 通过修改配置文件的方式来修改name和Email配置：
> > 文件目录：C:/Users/{user}/.gitconfig
> 打开文件进行修改即可
#### 删除相应配置：
```
git config --global --unset user.name
```
####git push origin master 时，每次都要输入用户名和密码，解决方案（最简）：
```
git config --global credential.helper store
```
> 输入上方这一句后，第一次提交还需要输入验证信息，以后都不用了（因为第一次输入的信息会被保存到一个文件中，下次直接调用了）
> <span style="background:yellow">这一步会在用户目录下生成文件.git-credential记录用户名密码的信息。</span>
[git push 无需验证的详细原理](https://blog.csdn.net/harryptter/article/details/75452016)

-------

####创建本地分支并推送到远程仓库
```
git checkout -b local_branch//创建本地分支local_branch
git push origin local_branch:local_branch//将本地分支local_branch(前一个)推送到远程仓库，并命名为local_branch（后一个，也可以任意命名）
```
####查看远程分支
```
git branch -a
git branch//这是查看本地分支
```
####删除远程分支
```
git push origin -d branch_name
git push origin --delete branch_name//或者用这个，完整delete时是两个横杠
```
#####也可以推送一个空分支到要删除的远程分支(即用空分支覆盖远程分支)，也可以达到删除远程分支的目的
```
git push origin : branch_name
```
####切换当前操作的分支
```
git checkout branch_name
```

####拉取远程指定分支代码
<a href="https://blog.csdn.net/weixin_39800144/article/details/78205617">git clone 指定分支 拉代码</a>
```
git clone -b branch_name http://10.1.1.11/service/tmall-service.git
```




----
<a href="#top" class="btn btn-primary" style="margin-top:10px">Top</a>


