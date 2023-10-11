# MyGarden

## This is my secrete garden. 


Let's begin with the git commands as the very starting point of building my garden!

ref:

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

    The satging area is a cache to store the things you added but not commited yet.

    The worksapce is the file directory that has ".git" associated with it.

    The stash is a place to hide modifications while you work on something else.

    NOTE: The area nearest to remote repo is the local repo, and you can only push things from LOCAL REPO to the remote repo, not from anywhere else.

    2.1. fetch
    
    it means to download info from remote to local repo.

    command: 

        git fetch origin

    Note that "pull" is not operated at your local repo, the "destination" of "pull" is at your workspace. And "git pull" is a shorhand for "git fetch" followed by "git merge FETCH_HEAD".

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

    ? what does it mean by origin/HEAD? why is it different from origin/min?

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

    recall the four areas, think about how the local repo and workspace are located from each other. remenber that there is "index" also related closely with both local repo and workspace.

    however, when the "data flow" is from remote and finally to your workspace, you may observe that the local repo does a lot of things directly to workspace, not to index. while, if you want to finally push your local stuff to remote repo, you always add to index when finished editing in workspace, and then commit to local repo, i.e., for this data flow direction, workspace usually does not directly contact the local repo.

    3.1. hard reset

        git reset --hard REMOTE_NAME/BRANCH_NAME

    Reset local repo and working tree to match a remote-tracking branch. Use reset ‑‑hard origin/main to throw away all commits to the local main branch. Use this to start over on a failed merge.

    3.2. branch

    create a new branch and switch to it:

        git checkout -b NEW_BRANCH_NAME

    switch to an existing branch:

        git checkout BRANCH_NAME

    or equivalently:

        git switch BRANCH_NAME

    if you have something not commited on the current branch, git won't let you to switch to another branch. Only if you have committed it, you are allowed to switch branch. and this may help you to understand that the branch operations are taken place in local repo (and results in your workspace change).

    3.3. merge

    merge current branch with another branch by:

        git merge ANOTHER_BRANCH

    this command will merge and commit. if you do not want to commit, use this command:

        git merge ANOTHER_BRANCH --no-commit

    as a result, it is "added" but no commited yet, i.e., the index is added to the staging (index) area.

    NOTE: the thing following "git merge" can also be a commit (i.e. not limited to branch).

    dealing with merge conflict:
    
    *TO DO*

    https://www.runoob.com/git/git-branch.html

    3.4. rebase

        git rebase UPSTREAM_BRANCH

     Reverts all commits since the current branch diverged from UPSTREAM_BRANCH, and then re-applies them one-by-one on top of changes from the HEAD of UPSTREAM_BRANCH.

    for more info, refer to: https://git-scm.com/docs/git-rebase

    3.5. cherry-pick

        git cherry-pick COMMIT
    
    Integrate changes in the given COMMIT (from another branch) into the current branch. (you should first use "git log" to find out the commit id on "another branch".)

    note that this will also create a commit (the commit you cherry-picked becomes a new commit to your current branch).

    for more information, refer to: https://blog.csdn.net/a1056244734/article/details/112908080

    3.6. revert a commit

        git revert COMMIT

    Reverse commit specified by COMMIT and commit the result. i.e., withdraw a commit and commit on the withdrawal. 
    
    NOTE: This requires your working tree to be clean (no modifications from the HEAD commit).






frequently used commands:

git add

git add -i

git commit -s -m

git commit --amend



git stash

git checkout filename

git log

git restore

git tag

 
