# MyGarden

## This is my secrete garden. 

Let's begin with the git commands as the very starting point of building my garden!

1. Create a repo and sync the local with remote:

    There are two ways of creating a repo, one is to create it at Github and then clone it to your local workspace; the other is to create a repo locally and then push it to remote (for example, Github).

    In the first way:

    Creating the repo is simple, no discussion needed. Then Clone it to any local address you like (the code is always provided by Github). 
(The directory you save the cloned repo usually should not have ".git"; the cloned repo itself has git, so, usually you don't want another git outside the repo.)
    Then you can add, commit and push from local to remote.

    if your local branch is called "main", the command to push is:
    git push origin -u main


    In the second way:
    
    you still need to create a repo at remote end (github). once you created it, github shows the URL of the repo.
    this case is suitable for the situation that you have a lot of things in local; and you have git for the local repo.
    then you can "link" the remote repo to the local repo by this command:

    git remote add origin git@github.com:JaneWen23/REPONAME.git

    the "origin" is just a nickname of the URL "git@github.com:USERNAME/REPONAME.git"; 

    if you set to a wrong URL, update it by:

    git remote set-url origin git@github.com:JaneWen23/REPONAME.git

    you can check whether changed successfully by:

    git remote -v

    if it is correct, then you can just push the local files to the remote repo as usual.

 

frequently used commands:

git add

git add -i

git commit -s -m

git commit --amend

git push origin -u main

git fetch origin

git pull

git stash

git checkout <filename>

git checkout -b

git rebase

git merge


 
