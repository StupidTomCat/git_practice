## git总结

[TOC]

### 小试牛刀

- git使用https协议，每次pull,push都要输入密码；使用git协议，使用ssh秘钥，可以省去每次输密码
- github与gitlab的区别
github如果不是开源项目的话是需要付费使用，所以选择使用gitlab，gitlab用于搭建私服
- git remote 命令用于在远程仓库的操作（ssh协议）
[git remote](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E4%BD%BF%E7%94%A8)
  1. git remote add <shortname> <url> 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写(别名)：git remote add origin git@github.com:StupidTomCat/springcloud.git
  2.**添加**文件到本地库：git add；**提交**文件到本地库：git commit -m "msg(提交日志)"： git commit -m "springcloud-hystrix-dashboard流量监控"；可合并（git commit -am ""）
  3. 推送到远程仓库 git push <remote> <branch>：git push origin master
  ps:git push -u origin master:-u的用法，加了参数-u后，以后即可直接用git push 代替git push origin master

### 分支简介

Git 鼓励在工作流程中频繁地使用分支与合并，哪怕一天之内进行许多次。 Git 保存的不是文件的变化或差异，而是一系列不同时刻的 **快照(snapshot)** ，在进行提交操作时，Git 会保存一个提交对象，该提交对象包含了作者的姓名和邮箱、提交时输入的信息。Git 可以在需要的时候重现此次保存的快照。

#### 查看分支

`git branch`

\*表示当前分支

#### 创建新分支

`git branch testing  --testing为新分支名`

git branch 命令仅仅 **创建** 一个新分支，并不会自动切换到新分支中去。

ps：创建新分支的同时切换过去

通常在创建一个新分支后立即切换过去，这可以用 `git checkout -b <newbranchname>` 一条命令搞定。

#### 切换分支

`git checkout testing`

ps：分支切换会改变工作目录中的文件。 如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子， 如果 Git 不能干净利落地完成这个任务，它将禁止切换分支，撒意思呢，意思是你要留意你的工作目录和暂存区里那些还没有被提交的修改， 它可能会和你即将检出(切换)的分支产生冲突从而阻止 Git 切换到该分支。==**总结起来就是，切换之前记得把之前的提交掉。**==

#### 使用 `git log` 命令查看各个分支当前所指的对象

提供这一功能的参数是 `--decorate`

```console
git log --oneline --decorate
```

结果：

```
631cc75 (HEAD -> testing, master) 有一个lalala文件  --表示当前是testing分支，HEAD 指向当前所在的分支
```

#### 小结

新分支testing与master的关系

![效果](E:\资料笔记\git\testing与master.png)

分支这种效果是真的强大

ps：git兼容一些Linux命令

[参考文章](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%AE%80%E4%BB%8B)

### 分支的合并

#### 合并(含删除分支)

1. 有相同祖先的简单合并

![基于 `master` 分支的紧急问题分支hotfix branch](E:\资料笔记\git\hotfix.png)

```console
git checkout master
git merge hotfix
```

简单解释一下，iss53是之前正在工作的分支，master是线上的稳定版本分支，在iss53工作时遇到需要紧急修复的问题时，切换到master分支然后创建出hotfix分支，在问题修复后即可合并，也就是上面两步(切换回master，然后合并)。

合并之后如下,可以着手发布该修复了。：

![合并后](E:\资料笔记\git\hotfix2.png)

紧急问题的解决方案发布之后，回到被打断之前时的工作中(iss53)。 应该先删除 `hotfix` 分支，因为已经不再需要它了 —— `master` 分支已经指向了同一个位置。 可以使用带 `-d` 选项的 `git branch` 命令来删除分支：

`git branch -d hotfix`

最终结果：

![回到iss53后](E:\资料笔记\git\hotfix3.png)



2. 不同祖先的合并(三方合并)

   已经修正了 #53 问题，并且打算将工作合并入 `master` 分支,为此，需要合并 `iss53` 分支到 `master` 分支:

   ```console
   git checkout master
   git merge iss53
   ```

   `master` 分支所在提交并不是 `iss53` 分支所在提交的直接祖先:

   ![状态](E:\资料笔记\git\merging-1.png)

出现这种情况的时候，Git 会使用两个分支的末端所指的快照（`C4` 和 `C5`）以及这两个分支的公共祖先（`C2`），做一个简单的三方合并。Git 将此次三方合并的结果做了一个新的快照(C6)并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于他有不止一个父提交，效果如下：

![合并结果](E:\资料笔记\git\merging-2.png)



合并后就不再需要 `iss53` 分支了，删除这个分支。

`git branch -d iss53`

3. 合并冲突

   如果对 #53 问题的修改和有关 `hotfix` 分支的修改都涉及到同一个文件的同一处，在合并它们的时候就会产生合并冲突。`git merge`产生冲突时，用 `git status` 命令来查看那些因包含合并冲突而处于未合并（unmerged）状态的文件，然后打开这些包含冲突的文件然后手动解决冲突， 出现冲突的文件看起来像下面这个样子：

   ```html
   <<<<<<< HEAD:index.html
   <div id="footer">contact : email.support@github.com</div>
   =======
   <div id="footer">
    please contact us at support@github.com
   </div>
   >>>>>>> iss53:index.html
   ```

   “`=======`”上面为当前分支，下面为产生冲突的分支，分隔的是冲突内容，选择一个（应该也可以自己换成其他的），例如：

   ```html
   <div id="footer">
   please contact us at email.support@github.com
   </div>
   ```

   解决了所有文件里的冲突之后，对每个文件使用 `git add` 命令来将其标记为冲突已解决， 一旦暂存这些原本有冲突的文件，Git 就会将它们标记为冲突已解决，确定之前有冲突的的文件都已经暂存了，这时可以输入 `git commit` 来完成合并提交。

### 分支管理

1. ```console
   $ git branch -v
     iss53   93b412c fix javascript issue
   * master  7a98805 Merge branch 'iss53'
     testing 782fd34 add scott to the author list in the readmes
   ```

   查看每一个分支的最后一次提交

2. ```console
   $ git branch --merged
     iss53
   * master
   ```

   查看哪些分支已经合并到当前分支。名字前没有 `*` 号的分支通常可以使用 `git branch -d` 删除掉，因为已经将它们合并了，所以并不会失去任何东西。

3. `--merged` 与 `--no-merged` 这两个有用的选项可以查看已经合并或尚未合并到当前分支的分支。 

   ```console
   $ git branch --no-merged
     testing
   ```

   这里显示了其他分支。 因为它包含了还未合并的工作，尝试使用 `git branch -d` 命令删除它时会失败，如果真的想要删除分支并丢掉那些工作，可以使用 `-D` 选项强制删除它。

   ps：可以提供一个附加的参数来查看其它分支的合并状态而不必切换到它们。 例如，尚未合并到 `master` 分支的有哪些？

   ```console
   $ git checkout testing
   $ git branch --no-merged master
     topicA
     featureB
   ```

### 分支开发工作流（了解）

#### 长期分支

在 `master` 分支上保留完全稳定的代码——有可能仅仅是已经发布或即将发布的代码。 还有一些名为 `develop` 或者 `next` 的平行分支，被用来做后续开发或者测试稳定性——这些分支不必保持绝对稳定，但是一旦达到稳定状态，它们就可以被合并入 `master` 分支了。 这样，在确保这些已完成的主题分支（短期分支，比如之前的 `iss53` 分支）能够通过所有测试，并且不会引入更多 bug 之后，就可以合并入主干分支中，等待下一次的发布。

![长期分支](E:\资料笔记\git\branches-1.png)

 稳定分支的指针总是在提交历史中落后一大截，而前沿分支的指针往往比较靠前。

#### 短期分支

它被用来实现单一特性或其相关工作。

### 远程分支(注意是分支不是仓库，强调分支的的概念)

远程分支以`<shortname>/<branch>`的形式命名(在本地电脑这样显示，在github上是master那还是master)。 例如，如果想要看我最后一次与远程仓库 `origin` 通信时 `master` 分支的状态，可以查看 `origin/master` 分支。

1. `git ls-remote <remote>`

   显式地获得远程引用的完整列表(不知所云)，我自己试了一下，效果如下：

   ![远程分支1](E:\资料笔记\git\远程分支1.png)

   head应该指的是当前的位置在main

2. `git remote show <shortname>`

   见远程仓库的第4点

3. 克隆之后状态（git clone）

   ![克隆](E:\资料笔记\git\克隆1.png)

4. 如果我在本地的 `master` 分支做了一些工作，在同一段时间内有其他人推送提交到 `git.ourcompany.com` 并且更新了它的 `master` 分支，这就是说我们的提交历史已走向不同的方向。 即便这样，只要我保持不与 `origin` 服务器连接（并拉取数据），我的 `origin/master` 指针就不会移动。

   ![克隆2](E:\资料笔记\git\克隆2.png)

5. `git fetch <shortname>`

   从远程仓库抓取本地没有的数据，并且更新本地数据库，移动 `origin/master` 指针到更新之后的位置。

   ps：另见远程仓库第2点

   ![克隆3](E:\资料笔记\git\克隆3.png)

6. 添加另一个远程仓库(远程分支 `teamone/master`)

   运行 `git fetch teamone` 后，因为另一台服务器上现有的数据是 `origin` 服务器上的一个子集， 所以 Git 并不会抓取数据而是会设置远程跟踪分支 `teamone/master` 指向 `teamone` 的 `master` 分支。

   ![克隆4](E:\资料笔记\git\克隆4.png)

   ps：当抓取到新的远程分支时(假如叫lalala)，这种情况下，不会有一个新的 `lalala` 分支——只有一个不可以修改的 `origin/lalala` 指针。

7. 推送

   `git push <shortname> <branch>`

   本地的分支并不会自动与远程仓库同步——必须显式地推送想要分享的分支。 

   如：`git push origin serverfix`

   这其实是一条简化的命令，Git 自动将 `serverfix` 分支名字展开为 `refs/heads/serverfix:refs/heads/serverfix`， 那意味着，“推送本地的 `serverfix` 分支来更新远程仓库上的 `serverfix` 分支。”也可以运行 `git push origin serverfix:serverfix`， 它会做同样的事——也就是说==**“推送本地的 `serverfix` 分支，更新远程仓库的 `serverfix` 分支”**== ，可以通过这种格式来推送本地分支到一个命名不相同的远程分支。 如果并不想让远程仓库上的分支叫做 `serverfix`，可以运行 `git push origin serverfix:awesomebranch` 来将本地的 `serverfix` 分支推送到远程仓库上的 `awesomebranch` 分支。

8. 跟踪分支

   - 在本地创建一个同名的分支跟踪远程分支：

     `git checkout -b serverfix origin/serverfix`

     跟踪分支是与远程分支有直接关系的本地分支。 如果在一个跟踪分支上输入 `git pull`，Git 能自动地识别去哪个服务器上抓取、合并到哪个分支。

     ps:克隆一个仓库时，通常会自动地创建一个跟踪 `origin/master` 的 `master` 分支。

   - 另一种方式（效果跟上面一致）：

     ```console
     $ git checkout --track origin/serverfix
     Branch serverfix set up to track remote branch serverfix from origin.
     Switched to a new branch 'serverfix'
     ```

   - 如果你尝试切换的分支不存在且当前没有与远程分支同名的本地分支，那么 Git 就会为你创建一个跟踪分支：

     ```console
     $ git checkout serverfix
     Branch serverfix set up to track remote branch serverfix from origin.
     Switched to a new branch 'serverfix'
     ```

   - 将本地跟踪分支与远程分支设置为不同的名字

     ```console
     $ git checkout -b sf origin/serverfix
     Branch sf set up to track remote branch serverfix from origin.
     Switched to a new branch 'sf'
     ```

   - 设置已有的本地分支跟踪一个刚刚拉取下来的远程分支，或者想要修改正在跟踪的本地分支， 可以在任意时间使用 `-u` 或 `--set-upstream-to` 选项运行 `git branch` 来显式地设置。

     ```console
     $ git branch -u origin/serverfix
     Branch serverfix set up to track remote branch serverfix from origin.
     ```

   - 查看所有的跟踪分支

     ```console
     $ git branch -vv
       iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
       master    1ae2a45 [origin/master] deploying index fix
     * serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
       testing   5ea463a trying something new
     ```

     这会将所有的本地分支列出来并且包含更多的信息，如每一个分支正在跟踪哪个远程分支与本地分支是否是领先、落后或是都有。需要重点注意的一点是这些数字的值来自于你从每个服务器上最后一次抓取的数据。 这个命令并没有连接服务器。

9. ==**拉取**==

     当 `git fetch` 命令从服务器上抓取本地没有的数据时，它并不会修改工作目录中的内容。 它只会获取数据然后让你自己合并。 然而，有一个命令叫作 `git pull` 在大多数情况下它的含义是一个 `git fetch` 紧接着一个 `git merge` 命令。 如果有一个设置好的跟踪分支，不管它是显式地设置还是通过 `clone` 或 `checkout` 命令为你创建的，`git pull` 都会查找当前分支所跟踪的服务器分支， 从服务器上抓取数据然后尝试合并入那个**远程分支**(比如叫origin/master)。

     由于 `git pull` 的魔法经常令人困惑所以通常单独显式地使用 `fetch` 与 `merge` 命令会更好一些。

10. 删除远程分支

     你和你的协作者已经完成了一个特性， 并且将其合并到了远程仓库的 `master` 分支（开发完成），从服务器上删除 `serverfix` 分支：

     ```console
     $ git push origin --delete serverfix
     To https://github.com/schacon/simplegit
      - [deleted]         serverfix
     ```
    
    经测试，这个命令删除的是github上面的分支：
    
    ![d1](E:\资料笔记\git\delete1.png)
    
    ![d2](E:\资料笔记\git\delete2.png)
    
    ![d3](E:\资料笔记\git\delete3.png)

### git的三个区域：

- **Working Tree** 当前的工作区域
- **Index/Stage** 暂存区域，和git stash命令暂存的地方不一样。使用git add xx，就可以将xx添加近Stage里面
- **Repository** 提交的历史，即使用git commit提交后的结果（快照，也即head，通常，理解 head的最简方式，就是将它看做 **该分支上的最后一次提交** 的快照。）

1. `git reset`主要作用

   1)`git reset --soft HEAD~`

   `reset` 做的第一件事是移动 HEAD 的指向。如果 HEAD 设置为 `master` 分支（例如，你正在 `master` 分支上）， 运行 `git reset 9e5e6a4` 将会使 `master` 指向 `9e5e6a4`，前面有v1，v2两次提交的历史，如下图：

   ![reset](E:\资料笔记\git\reset1.png)

   使用 `reset --soft`，它将仅仅停在那儿（应该是什么也不做的意思），`HEAD~`（HEAD 的父结点），暂存区(Index)和工作目录不会改变。**==它本质上是撤销了上一次 `git commit` 命令。==**

   关于head：

   - HEAD 表示当前版本(应该是上次的提交位置，检验和那儿)
   - HEAD^ 上一个版本
   - HEAD^^ 上上一个版本
   - HEAD^^^ 上上上一个版本
   - 以此类推...

   可以使用 ～数字表示

   - HEAD~0 表示当前版本
   - HEAD~1 上一个版本    HEAD~1`指回退一个快照，可以简写为`HEAD~
   - HEAD^2 上上一个版本
   - HEAD^3 上上上一个版本
   - 以此类推...

   2)`git reset --mixed HEAD~`  (git  reset默认添加mixed参数)

   接下来，`reset` 会用 HEAD 指向的当前快照的内容来更新暂存区。

   ![reset2](E:\资料笔记\git\reset2.png)

   `--mixed` 选项，`reset` 将会在这时停止。 这也是默认行为，所以如果没有指定任何选项（在本例中只是 `git reset HEAD~`），这就是命令将会停止的地方。

   现在再看一眼上图，理解一下发生的事情：它依然会撤销一上次 `提交`，但还会 *取消暂存* 所有的东西。 于是，我们**==回滚到了所有 `git add` 和 `git commit` 的命令执行之前。==**

   3)`git reset --hard HEAD~`

   ​	将暂存区的内容更新到工作目录， 如果使用 `--hard` 选项，它将会继续这一步。

   ![reset3](E:\资料笔记\git\reset3.png)

   现在的情况是撤销了git add` 和 `git commit` 命令以及**工作目录**中的所有工作。

   ps：`--hard` 标记是 `reset` 命令唯一的危险用法，它也是 Git 会真正地销毁数据的仅有的几个操作之一。 其他任何形式的 `reset` 调用都可以轻松撤消，但是 `--hard` 选项不能，因为它强制覆盖了工作目录中的文件。 在这种特殊情况下，我们的 Git 数据库(git repository)中的一个提交内还留有该文件的 **v3** 版本， 我们可以通过 `reflog` 来找回它。但是若该文件还未提交，Git 仍会覆盖它从而导致无法恢复。

2. `git reset`的作用2（通过路径来重置）

   head应该不是上文的head了

   1) `git reset file.txt` （这其实是 `git reset --mixed HEAD file.txt` 的简写形式，head指的是上一次提交的快照)

   ![rp1](E:\资料笔记\git\reset-path1.png)

   作用与`git add`效果相反

   2) `git reset eb43bf file.txt` 

   不让 Git 从 HEAD 拉取数据，而是通过具体指定一个提交来拉取该文件的对应版本。

   ![rp2](E:\资料笔记\git\reset-path2.png)

3. `git reset`的作用3（压缩）（了解）

   项目中，第一次提交中有一个文件，第二次提交增加了一个新的文件并修改了第一个文件，第三次提交再次修改了第一个文件。 由于第二次提交是一个未完成的工作，因此想要压缩它。

   <img src="E:\资料笔记\git\reset压缩1.png" alt="r压缩1" style="zoom:80%;" />

   运行 `git reset --soft HEAD~2` 来将 HEAD 分支移动到一个旧一点的提交上（即你想要保留的最近的提交）：

   <img src="E:\资料笔记\git\reset压缩2.png" alt="r压缩2" style="zoom:80%;" />

   然后只需再次运行 `git commit`：

   <img src="E:\资料笔记\git\reset压缩3.png" alt="r压缩3" style="zoom:80%;" />

   查看历史，看起来有个 v1 版 `file-a.txt` 的提交， 接着第二个提交将 `file-a.txt` 修改成了 v3 版并增加了 `file-b.txt`。 包含 v2 版本的文件已经不在历史中了。

   ps：commit命令也有压缩

4. `reset` 与`checkout`的区别

   1)不带路径

   运行 `git checkout [branch]` 与运行 `git reset --hard [branch]` 非常相似，它会更新所有三个区域使其看起来像 `[branch]`，不过有两点重要的区别。

   首先不同于 `reset --hard`，**==`checkout` 对工作目录是安全的，它会通过检查来确保不会将已更改的文件弄丢。==** 其实它还更聪明一些。它会在工作目录中先试着简单合并一下，这样所有_还未修改过的_文件都会被更新。 ==**而 `reset --hard` 则会不做检查就全面地替换所有东西。**==

   第二个重要的区别是 `checkout` 如何更新 HEAD。 ==**`reset` 会移动 HEAD 分支的指向，而 `checkout` 只会移动 HEAD 自身来指向另一个分支。**==

   例如，假设我们有 `master` 和 `develop` 分支，它们分别指向不同的提交；我们现在在 `develop` 上（所以 HEAD 指向它）。 如果我们运行 `git reset master`，那么 `develop` 自身现在会和 `master` 指向同一个提交。 而如果我们运行 `git checkout master` 的话，`develop` 不会移动，HEAD 自身会移动。 现在 HEAD 将会指向 `master`。

   <img src="E:\资料笔记\git\reset-checkout.png" alt="rc" style="zoom:80%;" />

   2)带路径

   运行 `checkout` 的另一种方式就是指定一个文件路径，这会像 `reset` 一样不会移动 HEAD。 它就像 `git reset [提交的检验和] file` 那样用该次提交中的那个文件来更新暂存区，但是它也会覆盖工作目录中对应的文件。 它就像是 `git reset --hard [提交的检验和] file`， 这样对工作目录并不安全，它也不会移动 HEAD。

5. 小结

   下面的速查表列出了命令对区域的影响。 “HEAD” 一列中的 “REF” 表示该命令移动了 HEAD 指向的分支引用（指向一个提交的检验和），而 “HEAD” 则表示只移动了 HEAD 自身。 特别注意 *WD Safe?* 一列——如果它标记为 **NO**，那么运行该命令之前请考虑一下。

   |                             | HEAD | Index | Workdir | WD Safe? |
   | :-------------------------- | :--- | :---- | :------ | :------- |
   | **Commit Level**            |      |       |         |          |
   | `reset --soft [commit]`     | REF  | NO    | NO      | YES      |
   | `reset [commit]`            | REF  | YES   | NO      | YES      |
   | `reset --hard [commit]`     | REF  | YES   | YES     | **NO**   |
   | `checkout <commit>`         | HEAD | YES   | YES     | YES      |
   | **File Level**              |      |       |         |          |
   | `reset [commit] <paths>`    | NO   | YES   | NO      | YES      |
   | `checkout [commit] <paths>` | NO   | YES   | YES     | **NO**   |

### 基础命令

1. 在已存在目录中**初始化仓库**

   ```console
   git init
   ```

   该命令将创建一个名为 `.git` 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干(这个是git的内部原理)。 

2. 克隆现有的仓库

    ```
   git clone <url>
   ```

   这会在当前目录下创建一个名为 “libgit2” 的目录，并在这个目录下初始化一个 `.git` 文件夹，Git 克隆的是该 Git 仓库服务器上的几乎所有数据。

   ```console
   git clone https://github.com/libgit2/libgit2 mylibgit
   ```

   自定义本地仓库的名字mylibgit

3. 文件状态       

   工作目录下的每一个文件都不外乎这两种状态：**已跟踪** 或 **未跟踪**

   意思是工作目录下也可以放不被记录的文件，哦呵呵，这就很让人放心

   文件的状态：

   ![文件状态](E:\资料笔记\git\lifecycle.png)

   - 当前文件状态

     git status

     查看哪些文件处于什么状态

     文件 出现在 `Changes not staged for commit` 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。在 `Changes to be committed` 这行下面的，就说明是已暂存状态。

     ps：`git status -s` 命令或 `git status --short` 命令缩短状态命令的输出。新添加的未跟踪文件前面有 `??` 标记，新添加到暂存区中的文件前面有 `A` 标记，修改过的文件前面有 `M` 标记。 输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态。

   - 跟踪新文件/暂存已修改的文件

     ```console
     git add 文件名
     ```

     如果是新文件，成功之后该文件名在 `Changes to be committed` 这行下面的(需要使用git status)，就说明该文件是已暂存状态。如果此时提交，那么该文件在你运行 `git add` 时的版本将被留存在后续的历史记录中。`git add` 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。

     如果是已修改的文件，要暂存这次更新，需要运行 `git add` 命令。 这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“精确地将内容添加到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。

   - 忽略文件(了解)

     忽略自动生成的文件，比如日志文件、临时文件，无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。建一个名为 `.gitignore` 的文件，列出要忽略的文件的模式。

     文件 `.gitignore` 的格式规范如下：

     - 所有空行或者以 `#` 开头的行都会被 Git 忽略。
     - 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
     - 匹配模式可以以（`/`）开头防止递归。
     - 匹配模式可以以（`/`）结尾指定目录。
     - 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。

     glob 模式是指 shell 所使用的简化了的正则表达式。 星号（`*`）匹配零个或多个任意字符；`[abc]` 匹配任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 问号（`?`）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如 `[0-9]` 表示匹配所有 0 到 9 的数字）。 使用两个星号（`**`）表示匹配任意中间目录，比如 `a/**/z` 可以匹配 `a/z` 、 `a/b/z` 或 `a/b/c/z` 等。这跟正则表达式差不多啊，哦呵呵。

     例子：

     ```
     # 忽略所有的 .a 文件
     *.a
     
     # 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
     !lib.a
     
     # 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
     /TODO
     
     # 忽略任何目录下名为 build 的文件夹
     build/
     
     # 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
     doc/*.txt
     
     # 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
     doc/**/*.pdf
     ```

4. 查看已暂存和未暂存的修改(了解)

   可以知道具体修改了什么地方，并且回答这两个问题：当前做的哪些更新尚未暂存？ 有哪些更新已暂存并准备好下次提交？

   - `git diff`

     查看尚未暂存的文件更新了哪些部分，也就是修改之后还没有暂存起来的变化内容。

   -  `git diff --staged` 

     查看已暂存的将要添加到下次提交里的内容，将比对已暂存文件与最后一次提交的文件差异

5. 提交

   谨记：==提交时文件的版本是最后一次运行 `git add` 命令时的那个版本，而不是运行 `git commit` 时，在工作目录中的当前版本。==

   `git commit`

   这个好像会启动默认的文本编辑器vim

   `git commit -m "更新说明"`

   所以，每次准备提交前，先用 `git status` 看下，你所需要的文件是不是都已暂存起来了， 然后再运行提交命令 `git commit`

   ==**请记住，提交时记录的是放在暂存区域的快照。**== 任何还未暂存文件的仍然保持已修改状态，可以在下次提交时纳入版本管理。 每一次运行提交操作，都是对项目作一次快照，==**以后可以回到这个状态，或者进行比较。**==

   `git commit -a -m "更新说明"`

   加上 `-a` 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 `git add` 步骤 。

6. 移除文件

   - 未暂存（即未跟踪）（何苦用这个）

     `rm 文件名  --在未跟踪的文件列表还是会有的，所以需要下面的命令`

     `git rm 文件名` 

     在下一次提交时，该文件就不再纳入版本管理了

   - 删除之前修改过或已经放到暂存区的文件，数据不能被 Git 恢复。

     在上面的基础上使用强制删除选项 `-f`

   - 从 Git 仓库中删除，但仍然希望保留在当前工作目录中（让文件保留在磁盘，但是并不想让 Git 继续跟踪，因为忘记弄.gitignore文件了）

     `git rm --cached 文件名`

7.  移动文件(文件重命名)

   Git 并不显式跟踪文件移动操作。

   `git mv 原名 新名`

[参考文章](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%AE%B0%E5%BD%95%E6%AF%8F%E6%AC%A1%E6%9B%B4%E6%96%B0%E5%88%B0%E4%BB%93%E5%BA%93)

### 撤销操作(之前经常遇到)

1. `git commit --amend`

   有时候我们提交完了才发现漏掉了几个文件没有添加，或者提交信息写错了。 此时，可以运行带有 `--amend` 选项的提交命令来重新提交

   ```console
   $ git commit -m 'initial commit'
   $ git add forgotten_file
   $ git commit --amend
   ```

   提交后发现忘记了暂存某些需要的修改,加一句add，然后使用--amend，最终只会有一个提交——第二次提交将代替第一次提交的结果。

2. 取消暂存的文件(在暂存区的文件)

   `git reset HEAD 文件名`
   
   执行 git reset HEAD 以取消之前 git add 添加，但不希望包含在下一提交快照中的缓存。
   
   不加文件则会取消所有暂存区的文件
   
   <img src="E:\资料笔记\git\git_reset_head.png" alt="gsh" style="zoom:80%;" />
   
3. 需求做了一半告诉你需求有变化，不再做了，如果你此时还没有进行commit

    `git reset --hard `

   这会直接删除工作目录的文件（是个狠人）：

   <img src="E:\资料笔记\git\git_reset_hard.png" alt="grha1" style="zoom:80%;" />

   <img src="E:\资料笔记\git\git_reset_hard3.png" alt="grha3" style="zoom:80%;" />

   <img src="E:\资料笔记\git\git_reset_hard2.png" alt="grha2" style="zoom:80%;" />

### 远程仓库的使用

1. 查看远程仓库

   1)`git remote`

   ​	它会列出你指定的每一个远程服务器的简写

   ​	ps:如果是克隆的仓库，至少应该能看到 ==**origin**== ——这是 Git 给你==**克隆的仓库服务器的默认名字**==

   2)`git remote -v`

   ````
   git remote -v
   origin	https://github.com/schacon/ticgit (fetch)  --拉取
   origin	https://github.com/schacon/ticgit (push)  --推送
   cho45   https://github.com/cho45/grit (fetch)  --拉取
   cho45   https://github.com/cho45/grit (push)  --推送
   ````

   ​	会显示需要读写远程仓库使用的 Git 保存的==**简写**==与其对应的 URL。

2. 添加远程仓库

    1)`git remote add <shortname> <url>` 

   ​	添加一个新的远程 Git 仓库，同时指定一个方便使用的==**简写**==，使用==**简写**==代替==**url**==

   2)`git fetch <shortname>`

   ​	拉取仓库中有但你没有的信息，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

   3)`git clone <url>`

   ​	如果你使用 `clone` 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为==**简写**==。

   ​	默认情况下，`git clone` 命令会自动设置本地 master 分支跟踪克隆的远程仓库的 `master` 分支（或其它名字的默认分支）。

3. 推送到远程仓库

   `git push <shortname> <branch>`

   克隆时通常会自动帮你设置好后面两个字段，即：

   `git push origin master`

   ps：当你和其他人在同一时间克隆，他们先推送到上游然后你再推送到上游，你的推送就会毫无疑问地被拒绝。 你必须先抓取他们的工作并将其合并进你的工作后才能推送。

4. 查看某个远程仓库

    `git remote show <shortname>` 

   会列出远程仓库的 URL 与跟踪分支的信息：

   ![远程仓库的信息](E:\资料笔记\git\远程分支2.png)

5. 远程仓库的重命名与移除

    1)`git remote rename 原名 新名` 

   ​	比如原名叫origin，那么分支就会变成这样origin/master  ->  新名/master（当然不要用中文名）

   2) `git remote rm <shortname>` 

   ​	移除一个远程仓库

### 查看提交历史

1. `git log`

   不传入任何参数的默认情况下，`git log` 会按时间先后顺序列出所有的提交，最近的更新排在最上面，包括提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明。

2. 选项 `-p` 或 `--patch` 

   会显示每次提交所引入的差异， 可以限制显示的日志条目数量，例如使用 `-2` 选项来只显示最近的两次提交：

   `git log -p -2`

3. 选项`--pretty`

   这个选项可以使用不同于默认格式的方式展示提交历史。 这个选项有一些内建的子选项。 比如 `oneline` 会将每个提交放在一行显示，在浏览大量的提交时非常有用。 

   ```console
   git log --pretty=oneline
   ca82a6dff817ec66f44342007202690a93763949 changed the version number
   085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
   a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
   ```

4. 选项`--since` 和 `--until`

   下面的命令会列出最近两周的所有提交：

   `git log --since=2.weeks`

   该命令可用的格式十分丰富——可以是类似 `"2008-01-15"` 的具体的某一天，也可以是类似 `"2 years 1 day 3 minutes ago"` 的相对日期。

5. 用 `--author` 选项显示指定作者的提交，用 `--grep` 选项搜索提交说明中的关键字。

6. `-S`

   接受一个字符串参数，并且只会显示那些添加或删除了该字符串的提交。 假设你想找出添加或删除了对某一个特定函数的引用的提交，可以调用：

   `git log -S function_name`

### 别名

 `git config` 文件：

```console
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```

当要输入 `git commit` 时，只需要输入 `git ci`。

### 标签

Git 可以给仓库历史中的某一个提交打上标签，以示重要。 比较有代表性的是人们会使用这个功能来标记发布结点（ `v1.0` 、 `v2.0` 等等）

1. 列出标签

   `git tag`

   以字母顺序列出标签。

2. 创建标签

   附注标签包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息。 通常会建议创建附注标签，但是如果你只是想用一个临时的标签， 或者因为某些原因不想要保存这些信息，那么也可以用轻量标签(它只是某个特定提交的引用)。

   1)附注标签

   ​	`git tag -a v1.4 -m "my version 1.4"`

   ​	指定 `-a` 选项,`-m` 选项指定了一条将会存储在标签中的信息。 如果没有为附注标签指定一条信息，Git 会启动编辑器要求你输入信	息。

   ​	`git show v1.4`

   ​	可以看到标签信息和与之对应的提交信息

   2)轻量标签

   ​	`git tag v1.4-lw`

   ​	创建轻量标签，只需要提供标签名字

   3)对过去的提交打标签

   ​	历史记录：

   ```console
   $ git log --pretty=oneline
   15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
   a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
   0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
   6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
   0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
   4682c3261057305bdd616e23b64b0857d832627b added a todo file
   166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
   9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
   964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
   8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme
   ```

   ​	在 “updated rakefile”打标签，就在末尾指定提交的校验和（或部分校验和）：

   ​	`git tag -a v1.2 9fceb02`

   4)共享标签

   ​	`git push` 命令并不会传送标签到远程仓库服务器上，必须显式地推送标签到共享服务器上。

   ​	`git push origin <tagname>`

   ​	ps：`git push origin --tags  --一次性推送很多标签`

3. 删除标签

   1)`git tag -d <tagname>`

   ​	删除本地仓库上的标签

   2) `git push <shortname> :refs/tags/<tagname>` 

   ​	从远程仓库中移除标签，上面的含义是，将冒号前面的空值推送到远程标签名，从而高效地删除它。

   3)`git push <shortname> --delete <tagname>`

   ​	第二种更直观的删除远程标签的方式

4. 检出标签（查看某个标签所指向的文件版本，有副作用，略）

### 变基（了解，这个有坑）

1. 在 Git 中整合来自不同分支的修改主要有两种方法：`merge` （此处指三方合并）以及 `rebase`（变基）。

   这两种整合方法的最终结果没有任何区别，但是变基使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的， 但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。

   一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁

   ps：无论是通过变基，还是通过三方合并，整合的最终结果所指向的快照始终是一样的，只不过提交历史不同罢了。 变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。

2. 总的原则是，只对尚未推送或分享给别人的本地修改执行变基操作清理历史， 从不对已推送至别处的提交执行变基操作，这样，你才能享受到两种方式带来的便利。

