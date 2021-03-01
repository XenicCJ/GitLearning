#Giteveryday

##介绍

本文介绍20多个每日常用的Git命令。

Git用户大致可以分为四类。对于每一类用户都介绍一小套日常使用的命令。

* 个人开发者（独立）中的命令对每个使用Git的用户都很重要，即使是独立工作的使用者。
* 如果你与他人协同工作，你也会用到个人开发者（参与）中介绍的命令。
* 项目集成员需要在以上命令的基础上再多掌握一些命令。
* 管理数据内容的作为系统管理员的人需要掌握库管理员的命令。

##个人开发者（独立）

独立的个人开发者不与其他开发者交换开发内容，在一个单独的库中独立工作，会使用到如下命令。

* [git init](https://git-scm.com/docs/git-init)用于创建新库
* [git log](https://git-scm.com/docs/git-log)用于查看历史记录
* [git switch](https://git-scm.com/docs/git-switch)和[git branch](https://git-scm.com/docs/git-branch)用于切换分支
* [git add](https://git-scm.com/docs/git-add)用于管理索引文件
* [git diff](https://git-scm.com/docs/git-diff)和[git status](https://git-scm.com/docs/git-status)用于查看做了哪些没有提交的变更
* [git commit](https://git-scm.com/docs/git-commit)用于将修改提交到当前分支
* [git restore](https://git-scm.com/docs/git-restore)用于撤销修改
* [git merge](https://git-scm.com/docs/git-merge)用于合并本地分支
* [git rebase](https://git-scm.com/docs/git-rebase)用于处理临时分支
* [git tag](https://git-scm.com/docs/git-tag)用于标记已知的历史节点（比如历史提交）

###例子

####使用一个tar压缩项目作为新库的起始点。

	$ tar zxf frotz.tar.gz
	$ cd frotz
	$ git init
	$ git add . (1)
	$ git commit -m "import of frotz source tree."
	$ git tag v2.43 (2)
	
1. 将所有内容添加到当前目录。
2. 生成一个轻量的，未注明的标签。

####创建一个临时分支然后做开发。

	$ git switch -c alsa-audio (1)
	$ edit/compile/test
	$ git restore curses/ux_audio_oss.c (2)
	$ git add curses/ux_audio_alsa.c (3)
	$ edit/compile/test
	$ git diff HEAD (4)
	$ git commit -a -s (5)
	$ edit/compile/test
	$ git diff HEAD^ (6)
	$ git commit -a --amend (7)
	$ git switch master (8)
	$ git merge alsa-audio (9)
	$ git log --since='3 days ago' (10)
	$ git log v2.43.. curses/ (11)
	
1. 创建一个新的临时分支。
2. 撤回在curses/ux_audio_oss.c所做的修改。
3. 你需要告知Git新增了文件；删除或修改会在稍后进行`git commit -a`的会被捕获。
4. 查看你将要提交的内容。
5. 提交所有内容，就和你已经检查的内容一样。
6. 检查所有此次提交的变更以及上一次提交。
7. 修改之前的提交，添加所有新的修改，使用原始消息。
8. 切换到master分支。
9. 合并临时分支到主分支。
10. 查看三天内的提交记录。
11. 查看从v2.43标签开始的，curses/下的提交记录。

##独立开发者（参与）

参与项目的开发者需要学习如何用Git与其他开发者交流项目，同时也要掌握单独的独立开发者所需掌握的命令。

* [git clone](https://git-scm.com/docs/git-clone)从原项目串流拷贝到你本地的库。
* [git pull](https://git-scm.com/docs/git-pull)和[git fetch](https://git-scm.com/docs/git-fetch)从已连接的初始项目串流拷贝最新修改到本地库。
* [git push](https://git-scm.com/docs/git-push)分享本地库修改的内容，用CVS风格就是分享库的工作流。
* [git format patch](https://git-scm.com/docs/git-format-patch)准备电子邮件提交，如果使用Linux核心风格论坛工作流。
* [git send email](https://git-scm.com/docs/git-send-email)不影响邮件用户代理的情况下发送电子邮件。
* [git request pull](https://git-scm.com/docs/git-request-pull)创建修改的总结并让上游拉取。

###例子

####克隆上游项目进行工作。并将修改上传到上游。

	$ git clone git://git.kernel.org/pub/scm/.../torvalds/linux-2.6 my2.6
	$ cd my2.6
	$ git switch -c mine master (1)
	$ edit/compile/test; git commit -a -s (2)
	$ git format-patch master (3)
	$ git send-email --to="person <email@example.com>" 00*.patch (4)
	$ git switch master (5)
	$ git pull (6)
	$ git log -p ORIG_HEAD.. arch/i386 include/asm-i386 (7)
	$ git ls-remote --heads http://git.kernel.org/.../jgarzik/libata-dev.git (8)
	$ git pull git://git.kernel.org/pub/.../jgarzik/libata-dev.git ALL (9)
	$ git reset --hard ORIG_HEAD (10)
	$ git gc (11)

1. 从主分支创建并切换到一个新分支mine
2. 根据需求重复一次。
3. 从你的分支拉取修改内容，以master分支为参考。
4. 用电子邮件发送。
5. 回到主分支，准备查看有什么新变化
6. `git pull`从上游项目拉取修改并合并到当前分支。
7. 在拉取后，马上查看上游所做的修改，只看感兴趣的部分。
8. 从一个外部库中获取分支名称
9. 从一个特定的外部库的特定分支ALL拉取变更并合并
10. 撤销拉取
11. 清理上次撤销留下的垃圾对象。

####推送变更到另一个库

	satellite$ git clone mothership:frotz frotz (1)
	satellite$ cd frotz
	satellite$ git config --get-regexp '^(remote|branch)\.' (2)
	remote.origin.url mothership:frotz
	remote.origin.fetch refs/heads/*:refs/remotes/origin/*
	branch.master.remote origin
	branch.master.merge refs/heads/master
	satellite$ git config remote.origin.push \
		   +refs/heads/*:refs/remotes/satellite/* (3)
	satellite$ edit/compile/test/commit
	satellite$ git push origin (4)
	
	mothership$ cd frotz
	mothership$ git switch master
	mothership$ git merge satellite/master (5)
	
1. mothership的电脑上在你的home目录下有个frotz项目库；从这个库克隆以在satellite的电脑上初始化一个库。
2. 克隆会默认设置一些参数。这条语句使`git pull拉取mothership机器上的分支到本地的remotes/origin/*远程追踪分支上。
3. 安排`git push`推送所有本地分支到相关的mothership电脑上的分支。
4. push命令会将我们的工作暂存在remotes/satellite/*中，并远程追踪mothership电脑上的分支。你可以以此作为备用选项。同样的，你可以假设mothership从你这“拉取”了项目（在只能单方操作的时候很好用）。
5. 在mothership的机器上，合并satellite上完成的工作到主分支。

####从特定标签产生分支

	$ git switch -c private2.6.14 v2.6.14 (1)
	$ edit/compile/test; git commit -a
	$ git checkout master
	$ git cherry-pick v2.6.14..private2.6.14 (2)

1. 以一个有名的标签创建一个私有分支。
2. 将private2.6.14分支的所有修改合并到master分支，而不使用正规的merge命令。或者也可以用`git format-patch -k -m --stdout v2.6.14..private2.6.14 | git am -3 -k`命令

另一种相应的参与开发者提交的机制是使用`git request-pull`或pull-request（比如用在GIthub上的），以向主机提醒你提交的修改。

##项目集成员

在多人参与的开发项目中如果有人专门作为项目集成员，那么他负责从其他开发者那里接收、检查、集成变更，然后让其他项目成员来获取和使用变更。

这部分内容也适用于负责`git request-pull`或在Github项目中负责整合项目内容的人员。库的分支助理也同时担当开发者和项目集成员的角色。

* [git am](https://git-scm.com/docs/git-am)从项目参与者接收电子邮件形式的补丁。
* [git pull](https://git-scm.com/docs/git-pull)从信任的项目负责人拉取合并变更。
* [git format patch](https://git-scm.com/docs/git-format-patch)做向项目参与者发送相关补丁的准备。
* [git revert](https://git-scm.com/docs/git-revert)撤销提交。
* [git push](https://git-scm.com/docs/git-push)发布最新的补丁。

###例子

###项目集成员普通的一天

	$ git status (1)
	$ git branch --no-merged master (2)
	$ mailx (3)
	& s 2 3 4 5 ./+to-apply
	& s 7 8 ./+hold-linus
	& q
	$ git switch -c topic/one master
	$ git am -3 -i -s ./+to-apply (4)
	$ compile/test
	$ git switch -c hold/linus && git am -3 -i -s ./+hold-linus (5)
	$ git switch topic/one && git rebase master (6)
	$ git switch -C seen next (7)
	$ git merge topic/one topic/two && git merge hold/linus (8)
	$ git switch maint
	$ git cherry-pick master~4 (9)
	$ compile/test
	$ git tag -s -m "GIT 0.99.9x" v0.99.9x (10)
	$ git fetch ko && for branch in master maint next seen (11)
	    do
		git show-branch ko/$branch $branch (12)
	    done
	$ git push --follow-tags ko (13)
	
1. 查看项目状态，查看你是否正在做某些工作。
2. 查看有哪些没有合并到主分支的分支。比如一些正在处理合并的分支，如`maint`,`next`和`seen`。
3. 阅读邮件，保留可应用的和未来会使用的内容。
4. 应用需要立刻应用的新修改，在项目集成员的签名下进行操作。
5. 根据需要创建临时分支并应用新修改，也是以项目集成员的签名进行操作。
6. 将内部的临时分支变基合并到主分支或单独拉出作为稳定分支。
7. 开辟一个新分支`seen`。
8. 打包出正在开发的临时分支
9. 将重要更新应用到旧版本。
10. 创建一个标签
11. 确保主分支没有在打补丁后重复提交更新。
12. 在`git show-branch`的输出中，`master`分支应该有所有`ko/master`含有的内容，`next`分支应该有所有`ko/next`分支所有的内容，等等。
13. 发布重要补丁以及指向发布历史的标签。

在这个例子中，`ko`便捷的指向了Git维护员的在kernel.org的库，看起来像这样：

	(in .git/config)
	[remote "ko"]
		url = kernel.org:/pub/scm/git/git.git
		fetch = refs/heads/*:refs/remotes/ko/*
		push = refs/heads/master
		push = refs/heads/next
		push = +refs/heads/seen
		push = refs/heads/maint

##系统管理员

系统管理员使用如下工具来设置和维护开发人员对库的通行许可：

* gitolite, gerrit code review, cgit等

###例子

####假设/etc/services有以下内容

	$ grep 9418 /etc/services
	git		9418/tcp		# Git Version Control System
	
####运行git-daemon来服务来自inetd的/pub/scm

	$ grep git /etc/inetd.conf
	git	stream	tcp	nowait	nobody \
	  /usr/bin/git-daemon git-daemon --inetd --export-all /pub/scm

实际进行设置的命令应该在同一行。
	  
####运行git-daemon来服务来自xinetd的/pub/scm

$ cat /etc/xinetd.d/git-daemon
# default: off
# description: The Git server offers access to Git repositories
	service git
	{
		disable = no
		type            = UNLISTED
		port            = 9418
		socket_type     = stream
		wait            = no
		user            = nobody
		server          = /usr/bin/git-daemon
		server_args     = --inetd --export-all --base-path=/pub/scm
		log_on_failure  += USERID
	}
	
检查xinetd文档并设置，这来自一个Fedora系统。其他系统内容可能不一样。

####使用git-over-ssh来设置Git的pull/push仅限于开发者

比如使用`$ git push/pull ssh://host.xz/pub/scm/project`

	$ grep git /etc/passwd (1)
	alice:x:1000:1000::/home/alice:/usr/bin/git-shell
	bob:x:1001:1001::/home/bob:/usr/bin/git-shell
	cindy:x:1002:1002::/home/cindy:/usr/bin/git-shell
	david:x:1003:1003::/home/david:/usr/bin/git-shell
	$ grep git /etc/shells (2)
	/usr/bin/git-shell
	
1. 登录保护设置在/usr/bin/git-shell，它仅允许`git push`和`git pull`。用户需要ssh来授权登录。
2. 在很多分布中/etc/shells需要列出使用的登录保护。

####CVS风格分享库

	$ grep git /etc/group (1)
	git:x:9418:alice,bob,cindy,david
	$ cd /home/devo.git
	$ ls -l (2)
	  lrwxrwxrwx   1 david git    17 Dec  4 22:40 HEAD -> refs/heads/master
	  drwxrwsr-x   2 david git  4096 Dec  4 22:40 branches
	  -rw-rw-r--   1 david git    84 Dec  4 22:40 config
	  -rw-rw-r--   1 david git    58 Dec  4 22:40 description
	  drwxrwsr-x   2 david git  4096 Dec  4 22:40 hooks
	  -rw-rw-r--   1 david git 37504 Dec  4 22:40 index
	  drwxrwsr-x   2 david git  4096 Dec  4 22:40 info
	  drwxrwsr-x   4 david git  4096 Dec  4 22:40 objects
	  drwxrwsr-x   4 david git  4096 Nov  7 14:58 refs
	  drwxrwsr-x   2 david git  4096 Dec  4 22:40 remotes
	$ ls -l hooks/update (3)
	  -r-xr-xr-x   1 david git  3536 Dec  4 22:40 update
	$ cat info/allowed-users (4)
	refs/heads/master	alice\|cindy
	refs/heads/doc-update	bob
	refs/tags/v[0-9]*	david
	
1. 将开发者放到同一个git组。
2. 使分享的库允许被该组写入。
3. 使用/howto/文档中Carl创建的update-hook进行分支协议控制。
4. alice和cindy可以推送到主分支，只有bob可以推送到doc-update，david作为发布经理且是唯一可以创建和推送标签的人。