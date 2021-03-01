#gittutorial

##概要 

本文主要对git官网上[gittutorial](https://git-scm.com/docs/gittutorial)进行学习总结，主要内容有如何将新项目导入Git，对项目做修改，以及将修改与其他开发者分享。

如果你是需要用Git来获取其他项目进行学习或开发的话，可以参考git官网上 [The Git User’s Manual](https://git-scm.com/docs/user-manual)前两章的内容。

首先请注意，如果你想获取某个命令的文档，比如`git log --graph`，可以用如下列命令：

	$ man git-log
	
或者：

	$ git help log
	
使用第二条命令时，你可以使用你设定的文档浏览器来阅读。详情见[git help](https://git-scm.com/docs/git-help)命令的文档。

正式使用Git前你可以将邮箱地址和姓名录入到Git系统中，最简洁的命令如下：

	$ git config --global user.name "Your Name Comes Here"
	$ git config --global user.email you@yourdomain.example.com
	
##导入新的项目

假设你有一个用tar打包的初始项目project.tar.gz。你可以用如下命令将项目加入Git的版本管理：

	$ tar xzf project.tar.gz
	$ cd project
	$ git init
	
Git命令行会显示：

	Initialized empty Git repository in .git/
	
现在就初始化好了工作索引，你会注意到多出了一个叫.git的文件夹。

接下来，用命令告诉git将当前索引所有文件纳入版本管理：

	$ git add .
	
现在，所有文件就纳入了Git中的“索引”的临时空间了。你可以使用如下命令将“索引”中的所有文件提交到库中：

	$ git commit
	
之后会有信息提示说你已经将所有文件提交。这样就在Git里储存好第一个版本的项目了。

##修改项目

修改一些文件，然后将这些修改了的文件加入到索引中：

	$ git add file1 file2 file3
	
这样就已经准备好提交了。你可以用如下命令来检查将要提交的是什么内容：

	$ git diff --cached
	
上面的命令如果去掉了--cached的参数，那么就会显示所有已修改但是没有加入索引的信息。你也可以用`git status`命令来查看项目简要的状态：

	$ git status
	On branch master
	Changes to be committed:
	Your branch is up to date with 'origin/master'.
	  (use "git restore --staged <file>..." to unstage)
	
		modified:   file1
		modified:   file2
		modified:   file3
		
如果你接下来要做调整的话，就用上述命令来检查，然后将所有修改过的内容添加到索引。最后就可以提交啦：

	$ git commit
	
回车执行后，命令行会再次告诉你提交的内容，并且为项目记录一个新的版本。

如果不这样一步一不做的话，你也可以用下面的命令来一次性完成添加到索引和提交的过程：

	$ git commit -a
	
上述命令会一次性将所有修改过但不是新增的文件添加到索引并且提交。

关于提交备注信息的注意事项：虽然不是必须的，但是有一种备注信息的格式非常正规。以一行简短的标题（50字母以内）来概述做出的变化，空一行，然后详细描述做出的变化。空行之上的是标题，这行标题在Git中始终存在。比如`git format patch`将一次提交转化为电子邮件，将第一行作为标题，剩下的内容作为邮件内容。

##Git追踪内容而不是文件

不少版本控制系统提供了一个`add`命令告诉系统开始追踪一个新文件。而Git的`add`命令则执行一些更简单更强大的操作：`git add`命令同时作用于新文件和修改过的文件，对这些文件创建快照并在索引中记录这些变更内容，以准备好进行下一次提交。

##查看项目历史

在任何阶段都可以用如下命令查看项目历史：

	$ git log
	
如果想查看每一个步骤的变更内容，使用如下命令：

	$ git log -p
	
通常可以用如下命令来查看每次变更的概述：

	$ git log --stat --summary
	
	
##管理分支

一个Git库可以管理开发过程中的多个分支。如果叫创建一个名为"experimental"的分支，使用如下命令：

	$ git branch experimental
	
创建好这个分支后执行如下命令来查看所有存在的分支：

	$ git branch
	
结果如下：

	  experimental
	* master

"experimental"分支是你刚刚创建的，而"master"分支是系统为你自动创建的默认分支。星号标明了你当前所在的分支。使用如下命令可以切换分支到"experimental"分支：

	$ git switch experimental

这样就切换分支了。现在修改一个文件，提交变更，然后切换回主分支：

	(edit file)
	$ git commit -a
	$ git switch master
	
检查一下会发现刚才做的变更看不到了，因为变更提交到的是experimental分支，而目前所在分支是主分支。

现在可以提交一次新变更到主分支上：

	(edit file)
	$ git commit -a
	
目前，这两个分支的内容就不一样了，因为在这两个分支提交了不同的变更。现在可以用如下命令来将experimental分支合并到主分支上：

	$ git merge experimental

如果两个分支的变更没有冲突的话，就完成合并了。如果有冲突，在冲突的文件里会有标记；
	
	$ git diff

`git diff`命令可以用于查看这些冲突。一旦冲突的文件经过修改后解决了冲突就可以提交了：
	
	$ git commit -a
	
如下命令可以用图形化展示目前的项目历史。

合并好之后可以用如下命令来删除experimental分支：

	$ git branch -d experimental

分支使用起来简单高效，适用于开发实践。

##使用Git来合作

假设Alice创建了一个新项目，并储存在了Git库/home/alice/project中，同时Bob在同样一台电脑上也有home文件夹，并且Bob参与项目合作开发。

Bob首先进行如下操作：

	bob$ git clone /home/alice/project myrepo
	
这个操作从Alice的库中克隆了一份项目，并存放在新建的"myrepo"文件夹中。克隆项目与原始项目处于同等地位，拥有原始项目历史且独立的副本。

Bob接下来做了一些变更并且提交了：

	(edit files)
	bob$ git commit -a
	(repeat as necessary)
	
当他完成提交后，他告诉Alice从位于/home/bob/myrepo的库中拉取变更。Alice执行了下列命令：

	alice$ cd /home/alice/project
	alice$ git pull /home/bob/myrepo master

上述命令从Bob的主分支获取变更并合并到Alice当前分支中。如果Alice之前在自己的项目中做了变更，那么她可能需要手动来修复冲突。

`pull`命令做了两个操作：从目标库的分支中获取变更，然后将变更合并到当前库的当前分支。

注意，Alice此时应该保证她当前库的变更都已经在拉取操作之前提交了。如果Bob做出的变更和Alice在Bob拉取项目后做的未提交变更有冲突，那么Alice的库会用自身的工作树和索引来解决冲突，此时当前库的未提交变更会影响冲突解决的过程（Git将依然会获取变更，但也会拒绝合并--Alice这时需要移除所有本地未提交变更，然后再次获取Bob的变更以解决问题）

Alice可以先查看Bob所做的变更，而不进行合并。这时可以使用"fetch"命令，执行这个命令后Alice就获取到Bob所做变更的日志了，当然也需要使用一个特殊的标记"FETCH_HEAD"。这个过程使用如下命令来获取变更以及协助决定要不要拉取变更：

	alice$ git fetch /home/bob/myrepo master
	alice$ git log -p HEAD..FETCH_HEAD
	
即使在Alice的库有未提交变更，上述操作也是安全的。上述的"HEAD..FETCH_HEAD"是指显示所有分支路线可以到达"FETCH_HEAD"节点的变更，同时不显示所有分支路线可以达到"HEAD"的变更。Alice的库已经有了所有"HEAD"之前的变更，这样就可以查看Bob直到"FETCH_HEAD"所做的变更，而Bob的变更之前不在Alice的库中。

如果Alice想以图形化的方式查看Bob在项目产生分叉后的历史记录，她可以使用如下命令：

	$ gitk HEAD..FETCH_HEAD
	
上述命令中也使用了之前`git log`用到的两点表示范围的参数。

Alice可能想看Bob克隆库后双方分别做了哪些事。她可以用三点表示范围的参数而不是两点：
	
	$ gitk HEAD...FETCH_HEAD
	
上述命令意味着展现双方的库各自都做了哪些修改，不显示双方都有的记录。

注意，两点表示范围和三点表示范围的参数在`gitk`和`git log`中都可用。

在查看了Bob所做的工作后，如果不是急需的话，Alice可以继续工作而不从Bob那儿拉取修改。如果Bob的修改中有Alice正需要的，Alice可以将修改内容先提取出来，然后进行一次"pull"，最后将提取的工作内容放回去。

如果你是在一个联系紧密的小组里开发的话，通常不会对同一个库反复操作。通过定义远程库快捷方式来简化这个过程：

	alice$ git remote add bob /home/bob/myrepo
	
这样Alice就可以执行拉取的第一部分操作-`git fetch`而不合并到她自己的分支上：
	
	alice$ git fetch bob
	
这样就比上面执行拉取的较长的命令更方便了，而且定义远程库后拉取的修改会存放在单独的分支里，默认分支名称是`bob/master`。同样的，下列语句会显示Bob拉取分叉项目后所做的修改：
	
	alice$ git log -p master..bob/master
	
检查完毕Bob的修改后，Alice可以把这些修改合并到自己的主分支上：

	alice$ git merge bob/master
	
上述合并操作也可以用`git pull`来完成，命令如下：

	alice$ git pull . remotes/bob/master
	
注意，`git pull`命令会把拉取的修改合并到当前分支上，不管添加了什么参数，执行了其他什么命令。

然后Bob可以用如下命令来更新Alice最近所做的修改：

	bob$ git pull
	
注意此时Bob不需要在命令中提供Alice的库的地址，因为当Bob一开始克隆Alice的库的时候，系统已经记录了Alice的库的地址，而这个地址正是用于拉取命令的：

	bob$ git config --get remote.origin.url
	/home/alice/project
	
（`git clone`命令所设置的配置信息可以用`git config -l`来查看，[git config](https://git-scm.com/docs/git-config)的文档说明了各个参数的含义）

Git同时也储存了一份Alice主分支的纯净版本在分支"origin/master"下：

	bob$ git branch -r
	  origin/master
	  
如果Bob过后打算在一个不同的主机上工作，他依然可以用ssh协议进行克隆和拉取操作：

	bob$ git clone alice.org:/home/alice/project myrepo
	
同样的，Git有原生的协议，或者也可以用http协议，详情见[git pull](https://git-scm.com/docs/git-pull)的文档。

Git也可以在类似CVS的模式下使用，各个用户可以将修改推送到一个中心库中，详情见[git push](https://git-scm.com/docs/git-push)和[gitcvs migration](https://git-scm.com/docs/gitcvs-migration)的文档.

##浏览历史

Git的历史记录表现为一系列相关的提交记录。我们已经看到了`git log`命令可以列出这些提交记录。注意，每一条记录的首行给出了这次提交的名称：

		ommit c82a22c39cbc32576f64f5c6b3f24b99ea8149c7
	Author: Junio C Hamano <junkio@cox.net>
	Date:   Tue May 16 17:18:22 2006 -0700
	
	    merge-base: Clarify the comments on post processing.
	    
我们可以将提交的名称作为`git show`的参数来查看这次提交的详细信息：

	$ git show c82a22c39cbc32576f64f5c6b3f24b99ea8149c7
	
也有其他方式来表明想要查看的提交记录。你可以用提交名称的足够长的开头部分来唯一表示一次提交：

	$ git show c82a22c39c	# the first few characters of the name are
				# usually enough
	$ git show HEAD		# the tip of the current branch
	$ git show experimental	# the tip of the "experimental" branch
	
每一个提交记录通常有一个”双亲“提交记录，这个”双亲“提交记录是上一次提交后项目的状态：
	
	$ git show HEAD^  # to see the parent of HEAD
	$ git show HEAD^^ # to see the grandparent of HEAD
	$ git show HEAD~4 # to see the great-great grandparent of HEAD

注意，合并的提交可能有超过一个双亲提交记录：

	$ git show HEAD^1 # show the first parent of HEAD (same as HEAD^)
	$ git show HEAD^2 # show the second parent of HEAD

你也可以赋予提交单独的名称，如下：

	$ git tag v2.5 1b2e1d63ff
	
你可以用名称"v2.5"来标记名为1b2e1d63ff的提交。如果你想把这个新名称和其他开发者共享（比如定义一个发布版本号），你应该创建一个"tag"标签对象，可能要做标记，详情见[git tag](https://git-scm.com/docs/git-tag)的文档。

所有与具体提交版本号有关的命令都可以用别名。比如：

	$ git diff v2.5 HEAD	 # compare the current HEAD to v2.5
	$ git branch stable v2.5 # start a new branch named "stable" based
			 				# at v2.5
	$ git reset --hard HEAD^ # reset your current branch and working
							 # directory to its state at HEAD^
							 
小心使用上述例子中最后一个命令。它会删除所有从这次提交开始之后所有在该分支上的提交记录。如果这个分支是项目唯一分支，那么这些提交的修改就会遗失。同时，也不要在其他开发者也可见的公共分支上使用`git reset` 命令，因为它会强制其他开发者进行不必要的操作以删除历史记录。如果你想撤销已经提交的变更，使用`git revert`命令即可。

`git grep`命令可以用来搜索项目中任何版本包含的字符串，比如下列命令可以在v2.5版本中搜索所有”hello“字符串：

	$ git grep "hello" v2.5
	
如果省略提交的名称，`git grep`会在当前索引搜索所有文件：

	$ git grep "hello"
	
这比较快速。

有多种方式来一次表达多个的提交名称，以下是几个例子：

	$ git log v2.5..v2.6            # commits between v2.5 and v2.6
	$ git log v2.5..                # commits since v2.5
	$ git log --since="2 weeks ago" # commits from the last 2 weeks
	$ git log v2.5.. Makefile       # commits since v2.5 which modify
					# Makefile
					
你也可以给`git log`提交名称一个范围，范围的开始可以不是第二个的祖先。如果"stable"分支和"master"分支从一个共同的提交分叉，那么下列命令将会列出在master分支而不在stable分支的提交：

	$ git log stable..master
	
下列命令则列出在stable分支而不在master分支的提交：

	$ git log master..stable
	
`git log`有一个缺点：它只会将提交以列表形式展现。当提交历史在某处分叉而之后又合并了，那么`git log`所展现的顺序就没有意义了。

不少有多个贡献者的项目，比如Linux核心或Git本身，经常有合并操作，这时`gitk`命令在展现历史信息上就更合适。比如：

	$ gitk --since="2 weeks ago" drivers/
	
上述命令可以展现所有两周内在"drivers"文件夹修改过文件的提交记录。（注意，你可以改变gitk的字体，按住control键同时按-或+就行）

最后，大多数接受文件名作为参数的命令可以在提交名称后跟文件名来指明某个版本的某个文件：

	$ git diff v2.5:Makefile HEAD:Makefile.in
	
你也可以用`git show`命令来查看任何这样的文件：

	$ git show v2.5:Makefile

##下一步

本教程应该足以用来进行基本的分布式版本控制系统的操作了。不过，如果要完全理解Git的深度和强大，你需要理解Git基于的两个基本理念：

* 对象数据库是一个用来存储项目文件、目录、提交记录的更加精美的系统。
* 索引文件是目录树状态的缓存，用于创建提交，检查工作目录，以及处理合并过程中的多个目录树。

本教程的第二部分讲解了目标数据库，索引文件，以及其他一些零星的东西。这些内容对你掌握Git非常有益。你可以在[gittutorial-2](https://git-scm.com/docs/gittutorial-2)中找到这些内容。

如果你暂时不想继续第二部分，其他一些内容可能也可以让你感兴趣：

* [git format patch](https://git-scm.com/docs/git-format-patch),[git am](https://git-scm.com/docs/git-am)：这两个命令将一系列git提交内容转化为电子邮件，反之亦然，适合用于类似Linux核心这样深度依赖电子邮件的项目。
* [git bisect](https://git-scm.com/docs/git-bisect)：如果项目因为出现问题问题而回溯，寻找bug的一种方式是搜索历史以寻找出现问题的提交记录。Git bisect可以帮助你执行一次二进制查找来寻找对应的提交记录。这个命令即使在复杂的分支繁多的项目中也运行自如。
* [gitworkflows](https://git-scm.com/docs/gitworkflows)：提供建议工作流程的概览。
* [giteveryday](https://git-scm.com/docs/giteveryday)：经常使用的20个命令。
* [gitcvs migration](https://git-scm.com/docs/gitcvs-migration)：供cvs用户使用的Git指南。
