+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2017-02-20T11:23:14+08:00"
menu = "main"
title = "hugo deploy as submodule on github"

+++

本Blog就是基于Hugo的，由于担心这次的操作记录丢失，我特意写了这篇博客来记录如何使用这个东西。

首先根据文档 [gohugo](http://www.gohugo.io/tutorials/github-pages-blog/)中的[第二部分](http://www.gohugo.io/tutorials/github-pages-blog/#hosting-personal-organization-pages)：

以上文章介绍了如何部署的大概情况，按照步骤来就不会有问题的。

但是维护起来需要熟悉git submodule的概念与使用，我自以为用过git submodule就安心的使用起来了，可是始终不得其意。
最终还是被我搞定了。

首先每次下载 https://github.com/qianguozheng/blog-hugo.git 的时候，总是没有自动更新[子模块](https://github.com/qianguozheng/qianguozheng.github.io.git)

```
	git submodule update --init --recursive
```
[来自](http://blog.csdn.net/wangjia55/article/details/24400501)

然后是执行./deploy.sh的时候没有提交子模块。

这个问题是我偶然发现问题的，我当时并不在master分支，而是特定的分支，导致无法提交。
所以 checkout下，提交就好了

整个工作流程如下：

```
	git clone https://github.com/qianguozheng/blog-hugo.git
	cd blog-hugo;
	#下载子模块
	git submodule update --init --recursive
	
	cd public
	git checkout master
	cd ../
	
	#创建博客
	hugo new post/Golang-test.md
	
	#编辑博客等操作
	...
	
	#提交qianguozheng.github.io.git仓库
	./deploy.sh
	
	#提交blog-hugo仓库
	git add -A
	git commit -m "Update blog"
	git push origin master	

```
完美结束

## 结论

不懂就多做测试，总会解决的
