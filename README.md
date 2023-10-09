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

        git remote：列出当前仓库中已配置的远程仓库。

        git remote -v：列出当前仓库中已配置的远程仓库，并显示它们的 URL。

        git remote add <remote_name> <remote_url>：添加一个新的远程仓库。指定一个远程仓库的名称和 URL，将其添加到当前仓库中。

        git remote rename <old_name> <new_name>：将已配置的远程仓库重命名。

        git remote remove <remote_name>：从当前仓库中删除指定的远程仓库。

        git remote set-url <remote_name> <new_url>：修改指定远程仓库的 URL。

        git remote show <remote_name>：显示指定远程仓库的详细信息，包括 URL 和跟踪分支。

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

    for example,

        git push origin -u main

    What does the "-u" mean? why sometimes it works without the "-u"?
    
    The key is "argument-less git-pull". When you do a git pull from a branch, without specifying a source remote or branch, git looks at the branch.<name>.merge setting to know where to pull from. git push -u sets this information for the branch you're pushing.

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

    Update: 

    in addition to git pull this setting also affects default behavior of git push. If you get in the habit of using -u to capture the remote branch you intend to track, I recommend setting your push.default config value to upstream.
    
    git push -u <remote> HEAD will push the current branch to a branch of the same name on <remote> (and also set up tracking so you can do git push after that).






frequently used commands:

git add

git add -i

git commit -s -m

git commit --amend



git stash

git checkout <filename>

git checkout -b

git rebase

git merge


 
