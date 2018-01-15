# Git Cheat Sheet

### CREATE

Clone an existing repository: `$ git clone ssh://user@domain.com/repo.git`

Create a new local repository: `$ git init`

### LOCAL CHANGES

Changed files in your working directory: `$ git status`

Changes to tracked files: `$ git diff`

Add all current changes to the next commit: `$ git add .`

Add some changes in <file> to the next commit: `$ git add -p <file>`

Commit all local changes in tracked files: `$ git commit -a`

Commit previously staged changes: `$ git commit`

Change the last commit: `$ git commit --amend`

### COMMIT HISTORY

Show all commits, starting with newest: `$ git log`

Show changes over time for a specific file: `$ git log -p <file>`

Who changed what and when in <file>: `$ git blame <file>`

### BRANCHES & TAGS

List all existing branches: `$ git branch -av`

Switch HEAD branch: `$ git checkout <branch>`

Create a new branch based on your current HEAD: `$ git branch <new-branch>`

Create a new tracking branch based on a remote branch: `$ git checkout --track <remote/branch>`

Delete a local branch: `$ git branch -d <branch>`

Mark the currnet commit with a tag: `$ git tag <tag-name>`

---

To be continued...