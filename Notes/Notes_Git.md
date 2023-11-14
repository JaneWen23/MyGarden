# Git Commands

## ref:

https://ndpsoftware.com/git-cheatsheet.html#loc=remote_repo

1. Create a repo and "link" the local to the remote:

    There are two ways of creating a repo, one is to create it at Github and then clone it to your local workspace; the other is to create a repo locally and then push it to remote (for example, Github).

    1.1. In the first way:

    Creating the repo is simple, no discussion needed. Then Clone it to any local address you like (the code is always provided by Github). 
(The directory you save the cloned repo usually should not have ".git"; the cloned repo itself has git, so, usually you don't want another git outside the repo.)
    Then you can add, commit and push from local to remote.

    if your local branch is called "main", the command to push is:
    
        git push origin -u main


    1.2. In the second way:
    
    you still need to create a repo at remote end (github). once you created it, github shows the URL of the repo.
    this case is suitable for the situation that you have a lot of things in local; and you have git for the local repo.
    then you can "link" the remote repo to the local repo by this command:

        git remote add origin git@github.com:JaneWen23/REPONAME.git

    the "origin" is just a nickname of the URL "git@github.com:USERNAME/REPONAME.git"; 

    if you set to a wrong URL, update it by:

        git remote set-url origin git@github.com:JaneWen23/REPONAME.git

    you can check whether changed successfully by:

        git remote -v

    if it is correct, you can just push the local files to the remote repo as usual.

 
    以下是 git remote 命令的常见用法：

        git remote
    
    列出当前仓库中已配置的远程仓库。

        git remote -v
    
    列出当前仓库中已配置的远程仓库，并显示它们的 URL。

        git remote add <remote_name> <remote_url>
    
    添加一个新的远程仓库。指定一个远程仓库的名称和 URL，将其添加到当前仓库中。

        git remote rename <old_name> <new_name>
    
    将已配置的远程仓库重命名。

        git remote remove <remote_name>
    
    从当前仓库中删除指定的远程仓库。

        git remote set-url <remote_name> <new_url>
        
    修改指定远程仓库的 URL。

        git remote show <remote_name>
    
    显示指定远程仓库的详细信息，包括 URL 和跟踪分支。

2. Operations between remote repo and local repo:

    Now, you need to understand what is a local repo. sometimes what it means is not what you think it means. 

    there are 4 areas at your local file system, namely, stash (aka 存档库), workspace (aka worktree, 工作区, or aka checkout), index (aka staging area, 暂存区索引), and local repo (aka 本地版本库).

    The local repo is the directory named ".git".

    The staging area (index) is a cache to store the things you added but not committed yet.

    The workspace is the file directory that has ".git" associated with it.

    The stash is a place to hide modifications while you work on something else.

    NOTE: The area nearest to remote repo is the local repo, and you can only push things from LOCAL REPO to the remote repo, not from anywhere else.

    2.1. fetch
    
    it means to download info from remote to local repo.

    command: 

        git fetch origin

    Note that "pull" is not operated at your local repo, the "destination" of "pull" is at your workspace. And "git pull" is a shorthand for "git fetch" followed by "git merge FETCH_HEAD".

    2.1.1. pull

    one mre thing about git pull: if your local reo and remote repo are diverged, let's say origin/master has commit A--B--C and local/master has commit A--B--D, when you try git pull, it will stop and prompt an error:

        hint: You have divergent branches and need to specify how to reconcile them.
        hint: You can do so by running one of the following commands sometime before
        hint: your next pull:
        hint: 
        hint:   git config pull.rebase false  # merge
        hint:   git config pull.rebase true   # rebase
        hint:   git config pull.ff only       # fast-forward only
        hint: 
        hint: You can replace "git config" with "git config --global" to set a default
        hint: preference for all repositories. You can also pass --rebase, --no-rebase,
        hint: or --ff-only on the command line to override the configured default per
        hint: invocation.
        fatal: Need to specify how to reconcile divergent branches.

    git pull --rebase is roughly equivalent to

        git fetch
        git rebase origin/master
    
    i.e. your remote changes (C) will be applied before the local changes (D), resulting in the following tree
    
    A -- B -- C -- D

    if I use git pull --ff-only ? -- It will fail.

    and one more thing: make sure to stash or commit the unstaged changes before "git pull".



    2.2. push

    update the server (remote) with your commits (in local repo). here may exist a problem that there are many branches in the local repo and you need to specify which to push.

    for example, push main branch to origin:

        git push origin -u main
    
    or another example, push another branch to origin:

        git push origin another_branch

    What does the "-u" mean: It means to set up the upstream repo.
    
    why sometimes it works without the "-u":
    
    The key is "argument-less git-pull". When you do a git pull from a branch, without specifying a source remote or branch, git looks at the "branch.name.merge" setting to know where to pull from. git push -u sets this information for the branch you're pushing.

    you just need to specify the upstream once for your branch. after that, you can omit the "-u" and git will know what you really mean.

    [following is copied from internet:]

    To see the difference, let's use a new empty branch:

        git checkout -b test

    First, we push without -u:

        git push origin test
        git pull


        You asked me to pull without telling me which branch you
        want to merge with, and 'branch.test.merge' in
        your configuration file does not tell me, either. Please
        specify which branch you want to use on the command line and
        try again (e.g. 'git pull <repository> <refspec>').
        See git-pull(1) for details.

        If you often merge with the same branch, you may want to use something like the following in your configuration file:

        [branch "test"]
        remote = <nickname>
        merge = <remote-ref>

        [remote "<nickname>"]
        url = <url>
        fetch = <refspec>

        See git-config(1) for details.
    Now if we add -u:

        git push -u origin test
        Branch test set up to track remote branch test from origin.
        Everything up-to-date

        git pull
        Already up-to-date.
    Note that tracking information has been set up so that git pull works as expected without specifying the remote or branch.

    in addition to git pull, this setting also affects default behavior of git push. If you get in the habit of using -u to capture the remote branch you intend to track, I recommend setting your push.default config value to upstream.
    
    git push -u REMOTE_REPO_NAME HEAD will push the current branch to a branch of the same name on REMOTE_REPO_NAME (and also set up tracking so you can do git push after that).

    2.3. branch

    you can list the remote branches by
    
        git branch -r

    ? what does it mean by origin/HEAD? why is it different from origin/main?

    you can list the local branches by

        git branch

    you can delete a remote branch by

        git push origin --delete BRANCH_NAME

    or you can delete a remote branch on Github.
    
    note that your local branch will not be deleted!

    you can delete a local branch by

        git branch -d BRANCH_NAME

    2.4 soft reset

        git reset --soft HEAD^
    
    it means to undo the last commit, leaving its changes in the workspace, uncommitted. Does not touch the index file or the working tree at all. The only change it makes is to the head of the local repository. (and "^" means to be backward by "1 step"; "^^" means backward by two steps)


3. operations between local repo and workspace

    recall the four areas, think about how the local repo and workspace are located from each other. remember that there is "index" also related closely with both local repo and workspace.

    however, when the "data flow" is from remote and finally to your workspace, you may observe that the local repo does a lot of things directly to workspace, not to index. while, if you want to finally push your local stuff to remote repo, you always add to index when finished editing in workspace, and then commit to local repo, i.e., for this data flow direction, workspace usually does not directly contact the local repo.

    3.1. hard reset

        git reset --hard REMOTE_NAME/BRANCH_NAME

    Reset local repo and working tree to match a remote-tracking branch. Use reset ‑‑hard origin/main to throw away all commits to the local main branch. Use this to start over on a failed merge.

    this will remove all the things that uncommitted (for example, the changes in workspace, and the things you added but not committed).

    is there any going back from hard reset??

    Yes, use "git reflog". (*TO DO*) =======================

    (*TO DO*) by the way, what is the mixed reset?? =========

    3.2. branch

    create a new branch and switch to it:

        git checkout -b NEW_BRANCH_NAME

    switch to an existing branch:

        git checkout BRANCH_NAME

    or equivalently:

        git switch BRANCH_NAME

    if you have something not committed on the current branch, git won't let you to switch to another branch. Only if you have committed it, you are allowed to switch branch. and this may help you to understand that the branch operations are taken place in local repo (and results in your workspace change).

    3.3. merge branches or commits

    merge current branch with another branch by:

        git merge ANOTHER_BRANCH

    this command will merge and commit. if you do not want to commit, use this command:

        git merge ANOTHER_BRANCH --no-commit

    as a result, it is "added" but no committed yet, i.e., the index is added to the staging (index) area.

    NOTE: the thing following "git merge" can also be a commit (i.e. not limited to branch).

    dealing with merge conflict:
    
    *TO DO*

    https://www.runoob.com/git/git-branch.html

    3.4. rebase (???)

        git rebase UPSTREAM_BRANCH

     Reverts all commits since the current branch diverged from UPSTREAM_BRANCH, and then re-applies them one-by-one on top of changes from the HEAD of UPSTREAM_BRANCH.

    for more info, refer to: https://git-scm.com/docs/git-rebase

    3.5. cherry-pick a commit

        git cherry-pick COMMIT
    
    Integrate changes in the given COMMIT (from another branch) into the current branch. (you should first use "git log" to find out the commit id on "another branch".) (and you need to make sure your working tree is clean -- if you just added a file to the index, you need to commit it before you can cherry-pick.)

    note that this will also create a commit (the commit you cherry-picked becomes a new commit to your current branch).

    for more information, refer to: https://blog.csdn.net/a1056244734/article/details/112908080

    何时使用merge, 何时使用cherry-pick: 前者是整个分支上所有的修改都合并过来, 后者是单个的commit.

    cherry-pick 与 merge一起用, 会有副作用:

    Cherry Pick 之後再 Merge 會產生重複的 Commit;

    透過 Cherry Pick 建立的 Commit 將難以追溯完整修改歷程;
    
    所以, 如果可以 Merge，就別用 Cherry Pick;

    refer to:
    
    https://blog.darkthread.net/blog/git-cherry-pick-cons/


    3.6. revert a commit

        git revert COMMIT

    Reverse commit specified by COMMIT and commit the result. i.e., withdraw a commit and commit on the withdrawal. But the name "revert" is better than withdraw, because it is technically the reverse of the target commit. and, if you revert twice, you will get the same as "no revert taken", which is better described by "revert".
    
    NOTE: This changes your workspace (you withdrew the commit, then the workspace restores to the stage when you last committed), so it requires your working tree to be clean (no modifications from the HEAD commit).

4. operations from index to local repo and to workspace

    4.1 reset an index (of a file)

    this is different from the soft reset and hard reset. this is just to undo the "add" operation, i.e., the file in workspace is still there, but instead, not marked for commit.

    I don't want to show the command for this reset because I want to avoid the confusion. this kind of reset can be done via "add -i" followed by choosing "3: revert" option.

    4.2. status

        git status
    
   Displays: 

    paths that have differences between the index file and the current HEAD commit, 

    paths that have differences between the workspace and the index file, and 

    paths in the workspace that are not tracked by git.

    4.3. checkout file(s) or directory

        git checkout FILE(S)_OR_DIR

        取回兩個版本之前的檔案 git checkout HEAD~2 FILE_NAME
    
    this is to abandon the modifications in your workspace and replace the file or directory with the version you added or committed last time.

    and this operation is from index to workspace. how to understand it? if you added a file into index, there is nothing to checkout, because your worktree is clean (worktree being clean meaning no untracked files and no modification in tracked files). if you modified a file which was added to the index earlier, when you checkout that file, git will replace it by the version you last added, not last committed.

    NOTE: the command "checkout" is used in two ways, one is to create and (or) switch branch, note that this makes a difference in your workspace; the other is to replace your files in the workspace.

    4.4. commit
        
        git commit -s -m

    the "-s" means to add a tailer showing who signed-off this commit in the end of your commit log message.

        git commit --amend

    this means to replace your last commit with this commit. the commit id will be different from the last commit, however, your last commit will no longer exist.

    how to commit to two branches at once?
    
    you can switch branch and cherry-pick, or, 
    
    another solution is to use git worktree, ref: https://andrewlock.net/working-on-two-git-branches-at-once-with-git-worktree/#creating-a-working-tree-from-an-existing-branch


5. operations from workspace to index

    the operations are add, remove (rm), and move (mv). after the operation, your files are in "staged" status.

    5.1. add

    this should be very familiar to you; just remember to use "git add -i".

    5.2. remove 

        git rm FILE(S)

    Remove a file from the workspace AND the index.

    *case 1*: the file was committed before you want to remove: 

    in this case, the file will be removed from the workspace and the index, so, you cannot find the file in the working directory, AND you cannot use "checkout" to put it back.

    if you regret deleting it, you should first use command

        git restore --staged FILE_NAME

    this is to restore it from staged to unstaged, i.e., put the deletion operation back to workspace. 

    at this time, you still cannot see the file in the working directory, because what we "put back" is the "location" that the "deletion operation" is saved at (from index to workspace). now it is the same as you put the file into "trash can" from system UI. 

    now you can use this command to really restore the file:

        git restore FILE_NAME

    because what "restore FILE_NAME" does is to discard changes in your working directory. 

    The difference between "checkout a file" and "restore a file" (both are referred to the operation between index and workspace):

    "checkout a file" only works for *contents* of existed files, not for deleted files, or a file you just added (maybe by accident). (for an existed file, it makes no difference using either checkout or restore to discard changes in your working directory.)

    "restore a file" can restore the operations on the file, namely, create (not added yet) and delete (put into trash can). 

    *case 2*: the file is added by accident and not committed yet

    in this case, when you type "git rm FILE_NAME", there will be an error prompt out, saying that the has changes in staged area, so you cannot do this.

    if you type "git rm -f FILE_NAME" to force removal, then there is no going back.

    (in this case, if you just want to not track the file, you can use "add -i" and choose "3: revert".)

    *case 3*: a file or folder was in track but you don't want to track anymore, and you would like it kept in your local working directory:

    in this case, the command should be:

        git rm --cached FILE_NAME

    if you want to untrack a folder, the command should be:

        git rm --cached -r FOLDER_NAME

    you can see that this only removes the "cached" (i.e. staged) records; and it again shows that "rm" is an operation towards the index.

    *case 4*: you moved your files using system UI or IDE UI before telling git:

    in this case, git treats the new directories as untracked new directories and the old paths as deleted. you have to "git add" the new and "git rm" the old.
    
    5.3. move

    this is used when you want to change the file's location, or rename, while still track the file. if you just move the file using your system UI, you will see that git reats it as deleted and another file at somewhere untracked, which is stupid.

    the command is:

        git mv OLD_FILE NEW_FILE

    (if you moved a file to another location using "git mv", this operation cannot be tracked by "restore --staged", maybe it's because restore can take care of add and remove, not move.)

6. operations at workspace and stash

    6.1. clean
    
    This operation is at the workspace only. it removes the files that have never been tracked before. (if a file was tracked but currently not tracked, it will not be removed.)
    
    usages: (copied from internet)

        git clean -n

    是一次clean的演习, 告诉你哪些文件会被删除. 它不会真的删除文件, 只是一个提醒.

        git clean -f

    删除当前目录下没有track过的文件(recursively, starting from the current directory). 它不会删除.gitignore里面指定的文件夹和文件, 不论这些文件是否被track过.

        git clean -f PATH

    删除指定路径下的没有被track过的文件.

        git clean -xf

    与 -f 的区别是会删除 .gitignore 指定的文件和文件夹.    

        git clean -df

    删除当前目录下没有被track过的文件和文件夹(不是recursively).

    另外, git reset --hard 与 git clean -f 是一对好基友. 结合使用他们能让你的工作目录完全会退到最近一次commit的状态.

    git clean 对于刚编译过的项目也非常有用. 如, 它能轻易删除掉编译后生成的 .o 和 .exe 等文件. 这在打包发布一个release的时候非常有用.

    6.2. stash

    including push, pop and apply. these are operations between workspace and stash.

    *case 1*: you want to record the current state of the working directory and the index, but want to go back to a clean working directory:
    
    The command saves your local modifications away and reverts the working directory to match the HEAD commit:

        git stash push -m "MESSAGE"

    or, for quickly making a snapshot, you can omit both push and MESSAGE:

        git stash

    the "message" will be automatically filled. but it is *recommended* to add -m "MESSAGE" on your own.

    默认情况下，git stash会缓存下列文件 (然后工作区被恢复到最新一次commit)：

    添加到暂存区的修改（staged changes）

    Git跟踪的但并未添加到暂存区的修改（unstaged changes）
    
    但不会缓存以下文件：

    在工作目录中新的文件（untracked files）

    被忽略的文件（ignored files）

    And you can have a bunch of stashes, use

        git stash list

    to view them.

    *case 2*: you want to apply (one of) the stashes in the list to your workspace:

    it is better to add currently edited to index before apply any stashed items.

    use "git stash list" to find out which one you want to apply (identified by the "index number"). for example,

        git stash apply 0
    
    this will apply the latest in the stash; (if there is conflict, git will auto-merge them, and after you are done with the conflict, you should add the file to the index.)

    and this will not remove the stashed item after "apply". if you want to remove it, use:

        git stash drop 0

    or, there is a shortcut for applying then drop the *latest* stashed item:

        git stash pop

    but if there is a conflict, "pop" command will not drop the stashed item in case you will need it later.

    *case 3*: you dropped a stash by mistake:

    i.) if your terminal is still there and you can see the returned message when you dropped it, you should know the ID (a series of number and letter), which looks like

        *[main][~/Occupational/Techniques/MyGarden]$ git stash drop 3  
        Dropped refs/stash@{3} (fda98edfc2b078644abd734c0b4b5d9fb3433530)

    the ID is fda98edfc2b078644abd734c0b4b5d9fb3433530. then you can still apply it by

        git stash apply fda98edfc2b078644abd734c0b4b5d9fb3433530

    NOTE: this applies the dropped stash to your workspace, but does not bring the stash back. the stash does not re-appear in the stash list.

    ii.) if you do now know the ID, you can find the ID by

        git fsck --unreachable

    since you just dropped it, its ID may be the first (latest) one. copy the ID and see what exactly it is by this command:

        git show fda98edfc2b078644abd734c0b4b5d9fb3433530

    if it is the one you are looking for, just git stash apply it.

    finally, you can drop ALL the stashes by

        git stash clear

7. tag

    there are two kind of tags, one is lightweight, and the other is annotated.

    *lightweight*: 

    lightweight tag is usually used for a minor version. this is just a tag attached to the commit info. nothing more.

    to add a lightweight tag to your latest commit:

        git add TAG_NAME

    NOTE: the TAG_NAME should not contain any white spaces; and your tag should be *unique* among all tags, no matter they are either lightweight or annotated.

    to add a lightweight tag to an earlier commit:

    use "git log" to find the target commit ID, then:

        git add TAG_NAME COMMIT_ID

    *annotated*:

    annotated tag is usually used for a release. this tag is a "entire object" (???).

    to add an annotated tag to the latest commit:

        git tag -a TAG_NAME -m "additional message about the release"
    
    to add an annotated tag to a history commit:

        git tag -a TAG_NAME -m "additional message about the release" COMMIT_ID

    *applied to both:*

    to delete a tag:

        git tag -d TAG_NAME

    to push a tag (or tags) to remote repo (the command omitted "main"):

        git push origin --tags

    or

        git push origin TAG_NAME

    to delete a tag at remote repo (the command omitted "main"):

        git push origin --delete TAG_NAME


8. Displaying information

        git log --oneline --graph 以較清楚易讀的格式顯示簽入歷程

        git log --oneline

        git reflog

        git show

        git fsck 

9. ignore
    if you want to ignore a folder or file in the repo, you can open a terminal at the root directory of the repo, then type:

        touch .gitignore

    then type

        ls -a

    to show all the items, you should see the .gitignore file there. 
    
    then use vim to edit the .gitignore file: if you want to ignore a file at the path ROOT/test/file.txt, you just type 

        test/file.txt

    if you want to ignore a folder at path ROOT/folder, just type
    
        folder/
    
    note the "/";

    more: ignore items starting with "img":

        img*

    ignore the files with a specific extension name (let's say .md)

        *.md

    and don't forget to add the .gitignore file:

        git add .gitignore

    for more info, ref:

    https://www.freecodecamp.org/news/gitignore-file-how-to-ignore-files-and-folders-in-git/



 
10. GitFlow 流程規範：

    Master 分支 - 穩定隨時可上線，多會加上版號

    Develop 分支 - 開發主線，Feature 由此分支出去，改好再合併進來

    Hotfix 分支 - 從 Master 分支出來，改完合併回 Master 及 Develop

    Release 分支 - Dev 成熟時分支到 Release 做上線前最後測試，測試沒問題合併至 Master 及 Develop

    Feature 分支 - 由 Dev 分支出來，寫完再併合 Dev
