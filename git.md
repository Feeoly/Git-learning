1. 
PS C:\Users\zhaoy\Documents\gitLearn> git push
fatal: The current branch master has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin master

2.

PS C:\Users\zhaoy\Documents\gitLearn> git push --set-upstream origin master
fatal: HttpRequestException encountered.
   发送请求时出错。
To https://github.com/Feeoly/Git-learning.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/Feeoly/Git-learning.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

3.
PS C:\Users\zhaoy\Documents\gitLearn> git pull
warning: no common commits
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/Feeoly/Git-learning
 * [new branch]      master     -> origin/master
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> master

4.
PS C:\Users\zhaoy\Documents\gitLearn> git pull origin master
From https://github.com/Feeoly/Git-learning
 * branch            master     -> FETCH_HEAD
fatal: refusing to merge unrelated histories

5. reference: https://www.educative.io/edpresso/the-fatal-refusing-to-merge-unrelated-histories-git-error
Consider the following two cases that throw this error(fatal: refusing to merge unrelated histories):
You have cloned a project and, somehow, the .git directory got deleted or corrupted. This leads Git to be unaware of your local history and will, therefore, cause it to throw this error when you try to push to or pull from the remote repository.
You have created a new repository, added a few commits to it, and now you are trying to pull from a remote repository that already has some commits of its own. Git will also throw the error in this case, since it has no idea how the two projects are related.

The error is resolved by toggling the allow-unrelated-histories switch. After a git pull or git merge command, add the following tag:
PS C:\Users\zhaoy\Documents\gitLearn> git pull origin master --allow-unrelated-histories

6.
PS C:\Users\zhaoy\Documents\gitLearn> git commit --amend
hint: Waiting for your editor to close the file...
[master 25fb7d9] first commit
 Date: Mon Sep 21 21:05:43 2020 +0800
 1 file changed, 1 insertion(+)
 create mode 100644 index.txt
On branch master

7. fast-forward v.s. non-fast forward
There is another common situation where you may encounter non-fast-forward rejection when you try to push, and it is possible even when you are pushing into a repository nobody else pushes into. After you push commit A yourself (in the first picture in this section), replace it with "git commit --amend" to produce commit B, and you try to push it out, because forgot that you have pushed A out already. In such a case, and only if you are certain that nobody in the meantime fetched your earlier commit A (and started building on top of it), you can run "git push --force" to overwrite it. In other words, "git push --force" is a method reserved for a case where you do mean to lose history.

reference: https://alphagao.com/2017/07/30/what-is-fast-forward/
如果不理会错误警告用本地更新强制覆盖仓库，就是一次 no-fast-forward 更新，很明显，no-fast-forward 更新会导致记录丢失。

那么这种问题是如何发生的呢？比如有两个人都是从仓库的 master 分支克隆到本地，然后分别开发。master 本身有一个指针 HEAD 指向最后一次 commit 记录 commit-0 。A 先完成一个功能，并 push 到仓库，这次 commit 记为 commit-A，这也就是一次 fast-forward 更新，此时仓库的 master 分支的 HEAD 指针就指向了 commit-A。接下来 B 也完成了一个功能，要向仓库 push commit-B，如果没有做额外操作，肯定会出现上面的错误。

知道错误是如何发生的，就可以避免了。既然仓库有了更新，那么就要先把仓库的更新拉取到本地。这里有两种方式可以拉取：一是直接使用 git pull 命令，该命令会在拉取的同时会直接与本地对应分支进行合并，如果确信仓库的更新与本地不会发生冲突，那么可以直接使用。但是很可能 A 与 B 都对同一些文件做出了修改，那么必然导致冲突。不过既然知道会冲突也只能老老实实解决冲突了，不管是 fetch 先解决冲突在合并还是 pull 先合并再解决冲突，这个过程少不了的，除非你确定仓库的更新是没用的可以直接抛弃，就可以执行 git push -f 强制覆盖到仓库，这会导致仓库中某些记录丢失。
但如果不想丢掉 commit-A 的同时又不想与 commit-A 合并，B 想继续接着本地仓库工作，可以使用 git rebase origin/master ,表示将本地所有 commit 排在仓库 的 commit 记录之后。然后向仓库的 push 就会被接受。

但其实还有一种比较常见的出现 no-fast-forward 这种错误的情境，是在你向一个只有你自己可访问的仓库 push 的时候发生的。当你已经将一次 commit-A push 到仓库后，然后因为某些原因又使用了 git commit –amend 修改了 commit-A ,这个时候 commit-A 就变成了 commit-B，而此时本地仓库就没有关系 commit-A 的记录了，这个时候再次向仓库 push ，很明显，commit-B 无法与仓库的 commit-A 进行对接，所以出现了 no-fast-forward 错误。这种情况下其实也很好解决，如果你确定 commit-A 已经完全无用并且没有人将 commit-A 拉取到本地进行进一步开发之后，你就使用 git push -f 来覆盖仓库记录。之后，你就会永远丢失 commit-A 记录了。

而对比发现，我之所以会遇到本文开头的错误，就是因为之前使用了 git commit --amend 命令修改了已经 push 到仓库的 commit 的注释导致的。因此，一旦已经 push 到仓库，想要做出修改，就只能通过一次新的 commit 来完成对某次已经 push 到仓库的 commit 记录的修改了，可以参考 githug 52 关 revert。

8.
PS C:\Users\zhaoy\Documents\gitLearn> git rebase origin/master
First, rewinding head to replay your work on top of it...
Applying: add index.txt in dev1
.git/rebase-apply/patch:8: trailing whitespace.
I fall in love with YOU
warning: 1 line adds whitespace errors.
Using index info to reconstruct a base tree...
M       index.txt
Falling back to patching base and 3-way merge...
Auto-merging index.txt
CONFLICT (content): Merge conflict in index.txt
error: Failed to merge in the changes.
hint: Use 'git am --show-current-patch' to see the failed patch
Patch failed at 0001 add index.txt in dev1
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".

PS C:\Users\zhaoy\Documents\gitLearn> git push

To push the history leading to the current (detached HEAD)
state now, use

    git push origin HEAD:<name-of-remote-branch>

PS C:\Users\zhaoy\Documents\gitLearn> git push origin HEAD:master
Everything up-to-date

PS C:\Users\zhaoy\Documents\gitLearn> git branch                                                                                           Everything up-to-date  
* (no branch, rebasing dev1)
  dev1
  master

so do below:
git rebase master
git checkout -b feature_branch_2
git push origin feature_branch_2


MmMmMming:
https://stackoverflow.com/questions/30471557/git-push-master-fatal-you-are-not-currently-on-a-branch/30471627

MmMmMming:
https://stackoverflow.com/questions/8939977/git-push-rejected-after-feature-branch-rebase

MmMmMming:
https://wiki.jikexueyuan.com/project/githug-walkthrough/level-28.html
