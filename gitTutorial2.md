#gittutorial2

##概述

你在阅读本教程前应当先完成[gittutorial](https://git-scm.com/docs/gittutorial)的内容。

本教程的目标是介绍Git架构中两个基础组成部分-对象数据库和索引文件-以及提供读者所有理解剩余Git文档所需的内容。

##Git对象数据库

我们首先创建一个新项目并且创建一些新的历史记录：

	$ mkdir test-project
	$ cd test-project
	$ git init
	Initialized empty Git repository in .git/
	$ echo 'hello world' > file.txt
	$ git add .
	$ git commit -a -m "initial commit"
	[master (root-commit) 54196cc] initial commit
	 1 file changed, 1 insertion(+)
	 create mode 100644 file.txt
	$ echo 'hello world!' >file.txt
	$ git commit -a -m "add emphasis"
	[master c4d59f3] add emphasis
	 1 file changed, 1 insertion(+), 1 deletion(-)
	
问题来了，Git用来标记提交记录的7位hex码是什么？

我们在第一部分教程中看到了提交记录有类似的名字。Git历史记录中的所有对象都用40位的hex码代表的名字标记。这个名字是对象内容的SHA-1哈希码；这保证了Git不会多次储存同样的数据（因为对每一个数据都给予了独立的SHA-1名称），以及每个Git对象都不会发生变化（因为这样也会同时变更对象的SHA-1名称）。7位的hex码只是40位码的开头部分。缩写码和40位码功能相同，因为它们并没有歧义。

如果你按照如上步骤进行了一样的操作，那么你的提交名还是不同的SHA-1码，因为SHA-1码的生成会受到时间和配置中设定的使用者名字影响。

我们可以用`cat file`命令来询问Git这个特定的对象。注意要复制你自己库中的40位hex码，而不是例子中的。注意，你可以使用位数更少的缩写，就不用打全40位码了：

	$ git cat-file -t 54196cc2
	commit
	$ git cat-file commit 54196cc2
	tree 92b8b694ffb1675e5975148e1121810081dbdffe
	author J. Bruce Fields <bfields@puzzle.fieldses.org> 1143414668 -0500
	committer J. Bruce Fields <bfields@puzzle.fieldses.org> 1143414668 -0500
	
	initial commit
	
一个树可以指向一个或多个blob对象，每个blob对象对应一个文件。一个树也可以指向其他的树对象，这样就可以生成一个树状结构。你可以检查任何树的结构，通过使用`git is-tree`命令（足够长的SHA-1开头部分也可以用）：

	$ git ls-tree 92b8b694
	100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    file.txt

这样我们就可以看到这个树里有一个文件。SHA-1哈希码代表了这个文件的数据：

	$ git cat-file -t 3b18e512
	blob

一个“blob”只是文件数据，我们也可以用`git cat-file`来查看：

	$ git cat-file blob 3b18e512
	hello world

注意这是旧的文件数据；Git根据初始树命名的对象是根据第一次提交生成的目录状态的闪照。

所有这些以SHA-1命名的对象储存在Git库中：

	$ find .git/objects/
	.git/objects/
	.git/objects/pack
	.git/objects/info
	.git/objects/3b
	.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
	.git/objects/92
	.git/objects/92/b8b694ffb1675e5975148e1121810081dbdffe
	.git/objects/54
	.git/objects/54/196cc2703dc165cbd373a65a4dcf22d50ae7f7
	.git/objects/a0
	.git/objects/a0/423896973644771497bdc03eb99d5281615b51
	.git/objects/d0
	.git/objects/d0/492b368b66bdabf2ac1fd8c92b39d3db916e59
	.git/objects/c4
	.git/objects/c4/d59f390b9cfd4318117afde11d601c1085f241
	
这些文件的内容只是压缩的数据以及标记了它们长度和类型的头部。类型是blob或树或提交或一个标签。

最容易找到的提交记录是Head，我们可以在.git/HEAD里找到：

	$ cat .git/HEAD
	ref: refs/heads/master
	
如你所见，这告诉我们我们当前所在的分支，并且是通过在.git文件夹下命名一个文件来实现的，这个文件包含一个SHA-1名称并指向一个提交对象，我们可以用`git cat-file`来检查：

	$ cat .git/refs/heads/master
	c4d59f390b9cfd4318117afde11d601c1085f241
	$ git cat-file -t c4d59f39
	commit
	$ git cat-file commit c4d59f39
	tree d0492b368b66bdabf2ac1fd8c92b39d3db916e59
	parent 54196cc2703dc165cbd373a65a4dcf22d50ae7f7
	author J. Bruce Fields <bfields@puzzle.fieldses.org> 1143418702 -0500
	committer J. Bruce Fields <bfields@puzzle.fieldses.org> 1143418702 -0500
	
	add emphasis
	
这里的树对象代表了树的新状态：

	$ git ls-tree d0492b36
	100644 blob a0423896973644771497bdc03eb99d5281615b51    file.txt
	$ git cat-file blob a0423896
	hello world!

而父母对象则代表前一次提交：

	$ git cat-file commit 54196cc2
	tree 92b8b694ffb1675e5975148e1121810081dbdffe
	author J. Bruce Fields <bfields@puzzle.fieldses.org> 1143414668 -0500
	committer J. Bruce Fields <bfields@puzzle.fieldses.org> 1143414668 -0500
	
	initial commit

树对象是我们首先检查的，这个初始提交的特别之处是没有父母节点。

大多数提交记录只有一个父母节点，但是有多个父母节点的提交记录也很常见。有多个父母节点的提交记录意味着这是一次合并操作，这些父母节点指向被合并分支的头结点。

除了blob，树和3提交记录，仅剩的其他一种对象是“标签”，不过这里我们不讨论标签。详情请见[git tag](https://git-scm.com/docs/git-tag)的文档。

现在我们了解了Git如何使用对象数据库来表示项目的历史：

* 提交对象内容包含项目在历史中的一个快照，以及体现了各个提交对象如何关联的父母提交节点。
* 树对象表明一个单独的目录的状态，将包含目录名和文件数据的blob对象和包含子目录信息的树对象联系起来。
* blob对象包含文件数据而没有其他结构。
* 每个分支的头部提交对象储存在.git/refs/heads/目录下的文件里。
* 当前分支的名字储存在.git/HEAD中。

注意，很多指令将树作为一个参数。如我们所见，树可以用多种方式表达-SHA-1名称、指向树的提交名、头部节点指向树的分支名称等等-而且大多类似的指令可以接受任一表达方式。

在命令概述中，"tree-ish"有时用于表示这样的一类参数。

##索引文件

我们主要用来创建一次提交的命令是`git commit -a`, 这个命令创建一次提交并包含了所有你在工作树种做的修改。但是如果你只需要提交一部分修改了的文件呢？或只提交某些文件的某些修改内容？

如果我们了解一下提交到底是如何创建的，我们就会发现有更灵活的创建提交的方式。

继续在我们的test-project中进行演示。我们先对file.txt再做一些修改：

	$ echo "hello world, again" >>file.txt
	
这次我们不直接创建提交，我们先进行一些中间步骤，使用diff方法来跟踪发生了什么事：

	$ git diff
	--- a/file.txt
	+++ b/file.txt
	@@ -1 +1,2 @@
	 hello world!
	+hello world, again
	$ git add file.txt
	$ git diff

上面例子中最后一个diff没有结果输出，但是我们现在并没有创建提交，并且HEAD节点里没有包含我们刚刚做的修改：

	$ git diff HEAD
	diff --git a/file.txt b/file.txt
	index a042389..513feba 100644
	--- a/file.txt
	+++ b/file.txt
	@@ -1 +1,2 @@
	 hello world!
	+hello world, again

所以，`git diff`比较的并不是HEAD结点。diff进行比较的正是存放在.git/index的索引文件，要比较索引文件内容我们可以用`git is-files`命令来检查：

	$ git ls-files --stage
	100644 513feba2e53ebbd2532419ded848ba19de88ba00 0       file.txt
	$ git cat-file -t 513feba2
	blob
	$ git cat-file blob 513feba2
	hello world!
	hello world, again

所以我们的`git add`命令执行的是创建了一个新的blob对象，然后放了一个引用在索引文件里。如果我们再次修改文件，我们就可以看到新的修改在`git diff`的输出中体现出来了：

	$ echo 'again?' >>file.txt
	$ git diff
	index 513feba..ba3da7b 100644
	--- a/file.txt
	+++ b/file.txt
	@@ -1,2 +1,3 @@
	 hello world!
	 hello world, again
	+again?
	
使用正确的参数，`git diff`也可以展现出工作目录和最近一次提交的差异，或者目录与上一次提交的差异：

	$ git diff HEAD
	diff --git a/file.txt b/file.txt
	index a042389..ba3da7b 100644
	--- a/file.txt
	+++ b/file.txt
	@@ -1 +1,3 @@
	 hello world!
	+hello world, again
	+again?
	$ git diff --cached
	diff --git a/file.txt b/file.txt
	index a042389..513feba 100644
	--- a/file.txt
	+++ b/file.txt
	@@ -1 +1,2 @@
	 hello world!
	+hello world, again

不论什么时候，我们可以用`git commit`（不包含-a参数）命令来创建新提交，以及验证提交的只是储存在索引目录中的修改，而没有加入索引的修改是不会提交的：

	$ git commit -m "repeat"
	$ git diff HEAD
	diff --git a/file.txt b/file.txt
	index 513feba..ba3da7b 100644
	--- a/file.txt
	+++ b/file.txt
	@@ -1,2 +1,3 @@
	 hello world!
	 hello world, again
	+again?
	
所以`git commit`默认用索引来创建提交而不是工作树；-a参数告诉commit命令先将所有工作树的修改更新到索引。

最后，可以看看`git add`命令对索引的效果：

	$ echo "goodbye, world" >closing.txt
	$ git add closing.txt

效果就是在索引文件中添加了一个实体：

	$ git ls-files --stage
	100644 8b9743b20d4b15be3955fc8d5cd2b09cd2336138 0       closing.txt
	100644 513feba2e53ebbd2532419ded848ba19de88ba00 0       file.txt

你可以在`git cat-file`中看到，新的实体指向文件的目前的内容：

	$ git cat-file blob 8b9743b2
	goodbye, world

而`git status`命令可以快速获取项目这两部分得到情况：

	$ git status
	On branch master
	Changes to be committed:
	  (use "git restore --staged <file>..." to unstage)
	
		new file:   closing.txt
	
	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git restore <file>..." to discard changes in working directory)
	
		modified:   file.txt
		
因为目前closing.txt已经加入了索引，所以会在"Changes to be committed"中。因为file.txt的修改还在工作空间内，没有加入到索引，所以会在"changed but not updated"中。这时，如果运行`git commit`，closing.txt会被提交而file.txt不会提交。

同时，注意没有参数的`git diff`会展示file.txt的修改，但不会显示closing.txt的修改，因为在索引中closing.txt的版本和在工作目录的版本一样。

除了作为新提交的缓存区域，索引文件也在提取分支时作为对象数据库的一部分，用于处理合并操作过程中的树。详情请见[gitcore-tutorial](https://git-scm.com/docs/gitcore-tutorial)。

##接下来做什么？

现在你应该知道了所有阅读git命令所需的前置知识了；接下来可以学习[giteveryday(https://git-scm.com/docs/giteveryday)中的命令。你可以在[gitglossary](https://git-scm.com/docs/gitglossary)中找到所有不知道的术语。

[Git User's Manual](https://git-scm.com/docs/user-manual)提供了Git的更加综合的介绍。

[gitcvs-migration](https://git-scm.com/docs/gitcvs-migration)讲解了如何将一个CVS库导入Git，并展示了如何用CVS风格使用Git.

如果想看一些有趣的Git用法，见[howtos](https://git-scm.com/docs/howto-index)。

对于Git开发者，[gitcore-tutorial](https://git-scm.com/docs/gitcore-tutorial)介绍了Git的底层运作方式，比如提交操作。