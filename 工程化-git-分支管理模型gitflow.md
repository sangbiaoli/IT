## gitflow分支管理模型

gitflow的分支类型：
* master分支（1个）
* develop分支（1个）
* feature分支。同时存在多个。
* release分支。同一时间只有1个，生命周期很短，只是为了发布。
* hotfix分支。同一时间只有1个。生命周期较短，用了修复bug或小粒度修改发布。

在这个模型中，master和develop都具有象征意义。master分支上的代码总是稳定的（stable build），随时可以发布出去。develop上的代码总是从feature上合并过来的，可以进行Nightly Builds，但不直接在develop上进行开发。当develop上的feature足够多以至于可以进行新版本的发布时，可以创建release分支。

release分支基于develop，进行很简单的修改后就被合并到master，并打上tag，表示可以发布了。紧接着release将被合并到develop；此时develop可能往前跑了一段，出现合并冲突，需要手工解决冲突后再次合并。这步完成后就删除release分支。

当从已发布版本中发现bug要修复时，就应用到hotfix分支了。hotfix基于master分支，完成bug修复或紧急修改后，要merge回master，打上一个新的tag，并merge回develop，删除hotfix分支。

由此可见release和hotfix的生命周期都较短，master/develop虽然总是存在但却不常使用。

以上就是gitflow的基本概念了。下面是nvie（gitflow的提出者，一个荷兰人！） A successful Git branching model（发布于2010年月5日）一文的笔记。

![](git/git_gitflow-layout.png)

从右看起：

* 时间轴。
* feature（玫红）。主要是自己玩了，差不多的时候要合并回develop去。从不与master交互。
* develop（黄色）。主要是和feature以及release交互。
* release（绿色）。总是基于develop，最后又合并回develop。当然对应的tag跑到* master这边去了。
* hotfix（红色）。总是基于master，并最后合并到master和develop。
* master（蓝色）。没有什么东西，仅是一些关联的tag，因从不在master上开发。