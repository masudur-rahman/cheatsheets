# GIT Cheatsheet


## Git Configuration

`$ git config --global user.name "username"`

`$ git config --global user.email "email"`

`$ git config --list`


## Git Initializaion
`$ git clone <repo-url> <local-new-directory>` Clones the repository creating a new folder


#### Locally initialize and pull from remote repository
`$ git init` 
`$ git pull <repo-url>` "git fetch + git merge" - fetch and merge with the local file.


## Stage changes to commit
`$ git add .`

`$ git add *`

`$ git add -all`

`$ git add -A` Stage all files including deleted ones

`$ git add <file-name>`

`$ git add *.<extension>` Supports regular expressions


## Commit changes
`$ git commit -m "message"` usual message format - "C1:branch-name message"

`$ git commit --amend -m "message"` `--amend / -a` modify or replace last commit if not pushed

`$ git push origin <branch-name>`


## History
`$ git status` status of files

`$ git log` history of commits

`$ git log -p` history of commits with changes

`$ git log -p <number-of-logs>` certain number of commit history

`$ git diff`  shows the differences with previous commit


## Branching 
`$ git branch <branch-name>` create new branch

`$ git checkout <branch-name>` moves HEAD to the branch

`$ git checkout -b <branch-name>` create and checkout at a time

`$ git checkout -` toggle with previous branch

`$ git branch` show local branches

`$ git branch -r` show remote branches

`$ git branch -a` show all branches

`$ git branch -v` show all branches along with last commit

`$ git branch -d <branch-name>` delete branch

`$ git branch -D <branch-name>` force delete branch


#### Move branches using refs
`$ git checkout HEAD~2` only move head

`$ git branch -f master HEAD~3` move branch to third ancestor.



## Stashing 
`$ git stash save 'message'` save dirty works along with a message

`$ git stash list` shows the list of stash

`$ git stash apply` get back last stash

`$ git stash aplly stash@{1}` get second last stash

`$ git stash drop` remove the stash from the list

`$ git stash drop stash@{1}`

`$ git stash pop` get back the last stash & remove from the list


