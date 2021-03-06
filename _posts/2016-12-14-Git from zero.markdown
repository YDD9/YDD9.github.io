---
layout: post
title:  "Git from zero!"
date:   2016-12-14 14:49:39 +0100
comments: true  
categories: git
---

Now you have git downloaded and ready to start.

# Table of contents
1. [Git config](#gitconfig)
2. [Git credential](#token)
3. [Adding an existing project to GitHub using the command line](#addproj)
4. [Ignore files or directories](#ignore)
5. [Checking the Status of Your Files](#statuscheck)
6. [Get git repository](#gitpull)  
7. [Go back to specific commit](#goback)  
8. [Git push succeed, remote not updating](#missupdates)  
9. [Create a local branch and force push to a remote branch](#forceLocalbranchtoRemotebranch)
10.[Branch create and merge](#BranchMerge)  
10. [Checkout a remote branch](#remotebranch)
11. [git diff merge tool](#diffmergetool)
12. [git submodules](#gitsubmodules)
13. [git merge master update into branch](#mergeMasterBranch)



## Git config <a name="gitconfig"></a>
The first thing you should do when you install Git is to set your user name and email address.   
This is important because every Git commit uses this information, and it’s immutably baked into the commits you start creating:   

```
$ git config --global user.name "John Doe"     
$ git config --global user.email johndoe@example.com     
```   

Again, you need to do this only once if you pass the --global option, because then Git will always use that information for 
anything you do on that system. If you want to override this with a different name or email address for specific projects, 
you can run the command without the --global option when you’re in that project.

Checking Your Settings
If you want to check your settings, you can use the git config --list command to list all the settings Git can find at that point:  

```
$ git config --list    
user.name=John Doe    
user.email=johndoe@example.com      
color.status=auto     
color.branch=auto     
color.interactive=auto     
color.diff=auto   
...   
```    

You may see keys more than once, because Git reads the same key from different files (/etc/gitconfig and ~/.gitconfig, for example).  
In this case, Git uses the last value for each unique key it sees. You can also check what Git thinks a specific key’s value is by typing git config <key>:   

```
$ git config user.name   
```  

You can always edit the config file by
```
git config --global --edit
```

## Git credential <a name="token"></a>

### Creating GitHub Personal Access Token

Of course, you can use your GitHub username/password to authenticate, but there is a better approach - Personal Access Tokens which:

Could be easily revoked from GitHub UI;
Have a limited scope;
Can be used as credentials with Git commands (this is what we need).
[Use this GitHub guide for creating access tokens.](https://help.github.com/articles/creating-an-access-token-for-command-line-use/)

### Enabling Git credential store

Git doesn’t preserve entered credentials between calls. However, it provides a mechanism for caching credentials called Credential Store. To enable credential store we use the following command:

```
git config --global credential.helper store
```

The git credential cache runs a daemon process which caches your credentials in memory and hands them out on demand. So killing your git-credential-cache--daemon process throws all these away and results in re-prompting you for your password if you continue to use this as the cache.helper option.

You could also disable use of the git credential cache using `git config --global --unset credential.helper`. Then reset this and you would continue to have the cached credentials available for other repositories (if any). You may also need to do `git config --system --unset credential.helper` if this has been set in the system config file (eg: Git for Windows 2).

On Windows you might be better off using the manager helper `git config --global credential.helper manager`. This stores your credentials in the Windows credential store which has a Control Panel interface where you can delete or edit your stored credentials. With this store, your details are secured by your Windows login and can persist over multiple sessions. The manager helper included in Git for Windows 2.x has replaced the earlier *wincred* helper that was added in Git for Windows 1.8.1.1. A similar helper called winstore is also available online and was used with GitExtensions as it offers a more GUI driven interface. The manager helper offers the same GUI interface as winstore.

Extract from Windows manual detailing the Windows credential store panel:

Open User Accounts by clicking the Start button Picture of the Start button, clicking Control Panel, clicking User Accounts and Family Safety (or clicking User Accounts, if you are connected to a network domain), and then clicking User Accounts. In the left pane, click Manage your credentials.
source from http://stackoverflow.com/questions/15381198/remove-credentials-from-git


## Adding an existing project to GitHub using the command line <a name="addproj"></a>
Create a new repository on GitHub. To avoid errors, do not initialize the new repository with README, license, 
or gitignore files. You can add these files after your project has been pushed to GitHub. 

1.Open Git Bash.   
2.Change the current working directory to your local project. (pwd, cd tab, dir)   
3.Initialize the local directory as a Git repository.   
```
git init   
```  
4.Add the files in your new local repository. This stages them for the first commit.    
```
git add .   
```  
5.Adds the files in the local repository and stages them for commit. To unstage a file, use `git reset HEAD YOUR-FILE`.   
Commit the files that you've staged in your local repository.   
```
git commit -m "First commit"   
```    
Commits the tracked changes and prepares them to be pushed to a remote repository. To remove this commit and modify the file,   
use `git reset --soft HEAD~1` and commit and add the file again.   
6.In the Command prompt, add the URL for the remote repository where your local repository will be pushed. 'orgin' is the name of the remote repo.   
```
git remote add origin remote repository URL   
```    
Display the added remote `git remote -v`   
7.Push the changes in your local repository to GitHub.   
```
git push origin master   
```    
Pushes the changes in your local repository up to the remote repository you specified as the origin    
8.Changing a remote's URL. The `git remote set-url` command takes two arguments:  
a). An existing remote name. For example, origin or upstream are two common choices.    
b). A new URL for the remote. For example:   
  If you're updating to use HTTPS, your URL might look like: `https://github.com/USERNAME/OTHERREPOSITORY.git`    
  If you're updating to use SSH, your URL might look like: `git@github.com:USERNAME/OTHERREPOSITORY.git`     
9.Delete remote URLs by their names. As a side note, it should be pointed that there is     
also `git remote rm <name>` that deletes every remotes matching the given name, whatever the URLs.    


## Ignore files or directories <a name="ignore"></a>  


Create a file named .gitignore in your projects directory. Ignore directories by entering the directory name into the file dir_to_ignore/ (with a slash appended) or by cmd:

```
echo filename ignore.sqlite3 >> .gitignore
echo filename.log >> .gitignore

# check status, remove .pyc files and add .pyc in ignore list
git status

git rm -r --cached superlists/superlists/__pycache__

echo __pycache__ >> .gitignore
echo *.pyc >> .gitignore
```

below sample ignore: folders, A type of files, Single file, A filename with space and dash

```
Datafiles/1/
.git
.vscode

*.pyc

env.json
exe_cmd.bat
rundeploy.txt
log.txt

Datafiles/cf\ commands\ -\ sps\ perf.txt
```


PS: 

.gitignore will only ignore files that you haven't already added to your repository.

If you did a `git add .`, and the file got added to the index, .gitignore won't help you. You'll need to do `git rm -r -cached sites/default/settings.php` to remove it, and then it will be ignored. You can have a dry run `git add -n .`






## Checking the Status of Your Files <a name="statuscheck"></a>
The main tool you use to determine which files are in which state is the git status command.    
If you run this command directly after a clone, you should see something like this:    

```
$ git status   
On branch master   
Your branch is up-to-date with 'origin/master'.    
nothing to commit, working directory clean    
```   

This means you have a clean working directory – in other words, there are no tracked and modified files. Git also doesn’t see any untracked files,    
or they would be listed here. Finally, the command tells you which branch you’re on and informs you that it has not diverged from the same branch     
on the server. For now, that branch is always “master”, which is the default; you won’t worry about it here. Git Branching will go over branches and references in detail.    

To see what you’ve changed but not yet staged, type git diff with no other arguments:     

```
$ git diff   
diff --git a/CONTRIBUTING.md b/CONTRIBUTING.md    
index 8ebb991..643e24f 100644     
--- a/CONTRIBUTING.md    
+++ b/CONTRIBUTING.md     
@@ -65,7 +65,8 @@ branch directly, things can get messy.       
Please include a nice description of your changes when you submit your PR;      
if we have to read the whole diff to figure out why you're contributing    
in the first place, you're less likely to get feedback and have your changed     
-merged in.     
+merged in. Also, split your changes into comprehensive chunks if your patch is    
+longer than a dozen lines.    
```      

That command compares what is in your working directory with what is in your staging area. The result tells you the changes you’ve made that you haven’t yet staged.     
If you want to see what you’ve staged that will go into your next commit, you can use `git diff --staged`. This command compares your staged changes to your last commit:       

```
$ git diff --staged     
diff --git a/README b/README    
new file mode 100644    
index 0000000..03902a1    
--- /dev/null    
+++ b/README    
@@ -0,0 +1 @@    
+My Project    
```    

It’s important to note that git diff by itself doesn’t show all changes made since your last commit – only changes that are still unstaged.      
This can be confusing, because if you’ve staged all of your changes, git diff will give you no output.    


## Get git repository <a name="gitpull"></a>  
+ Clone the full repository  

You clone a repository with `git clone [url]`. For example, if you want to clone the Git linkable library called libgit2, you can do so like this:  
```
$ git clone https://github.com/libgit2/libgit2
```  
That creates a directory named “libgit2”, initializes a .git directory inside it, pulls down all the data for that repository, and checks out a working copy of the latest version. If you go into the new libgit2 directory, you’ll see the project files in there, ready to be worked on or used. If you want to clone the repository into a directory named something other than “libgit2”, you can specify that as the next command-line option:  

```
$ git clone https://github.com/libgit2/libgit2 mylibgit
```  

That command does the same thing as the previous one, but the target directory is called mylibgit.

+ Download code changes from repository  

Incorporates changes from a remote repository into the current branch. In its default mode, `git pull` is shorthand for `git fetch` followed by `git merge FETCH_HEAD`.  

```
git pull <remote name> <remote branch name>
git pull origin master
```  

PS: If any of the remote changes overlap with local uncommitted changes, the merge will be automatically cancelled and the work tree untouched. It is generally best to get any local changes in working order before pulling or stash them away with `git-stash`.  

+ Working on different branches  

List existing branches, the current branch will be highlighted with an asterisk *  

```
git branch --list
```   

Go to an existing branch

```
git checkout mybranch1
```   

Create a branch "mycranch2" and go to this branch immediately `git checkout -b mybranch2`.   

+ Download objects and refs from another repository  

Fetch branches and/or tags (collectively, "refs") from one or more other repositories, along with the objects necessary to complete their histories. Remote-tracking branches are updated (see the description of <refspec> below for ways to control this behavior).

By default, any tag that points into the histories being fetched is also fetched; the effect is to fetch tags that point at branches that you are interested in. This default behavior can be changed by using the --tags or --no-tags options or by configuring remote.<name>.tagOpt. By using a refspec that fetches tags explicitly, you can fetch tags that do not point into branches you are interested in as well.

git fetch can fetch from either a single named repository or URL, or from several repositories at once if <group> is given and there is a remotes.<group> entry in the configuration file. (See git-config).

When no remote is specified, by default the origin remote will be used, unless there’s an upstream branch configured for the current branch.  

The names of refs that are fetched, together with the object names they point at, are written to .git/FETCH_HEAD. This information may be used by scripts or other git commands, such as git-pull.  

Update the remote-tracking branches:  

```
$ git fetch origin
```   

The above command copies all branches from the remote refs/heads/ namespace and stores them to the local refs/remotes/origin/ namespace, unless the branch.<name>.fetch option is used to specify a non-default refspec.

Using refspecs explicitly:

```
$ git fetch origin +pu:pu maint:tmp
```  

This updates (or creates, as necessary) branches pu and tmp in the local repository by fetching from the branches (respectively) pu and maint from the remote repository.  

The pu branch will be updated even if it is does not fast-forward, because it is prefixed with a plus sign; tmp will not be.

Peek at a remote’s branch, without configuring the remote in your local repository:

```
$ git fetch git://git.kernel.org/pub/scm/git/git.git maint
$ git log FETCH_HEAD
```  

The first command fetches the maint branch from the repository at git://git.kernel.org/pub/scm/git/git.git and the second command uses FETCH_HEAD to examine the branch with git-log. The fetched objects will eventually be removed by git’s built-in housekeeping (see git-gc).


## Go back to specific commit<a name="goback"></a>

`git log` to find out guid number of your desired commit
`git checkout <guid>` to go back to that specific commit



## Git push succeed, remote not updating <a name='missupdates'></a>

Here's setup case, simple and everything setup by default. Locally only one default branch: master, on the github remote origin only one branch master as well.   
After committing changes locally and `git push origin master` failed due to remote version was newer(I changed one filename on github.com), so `git pull origin master` and enter your merge commit message. Pushed again the same error.   
Nightmare began, to FORCE a push `cf push -f origin master` on github.com I lost one file(the filename changed one) and newly committed local files were not pushed. Specify with `git add newfile` and committ and push, nothing added into github.com/project.git Newly local files were always not pushed to remote github.com/project.git.

solution: `git push origin HEAD:master`
[doc link](https://git-scm.com/docs/git-push)  
`git push [remoterepo] [src]:[dst]` HEAD used in git to prepresent [your last commit snapshot](https://git-scm.com/blog/2011/07/11/reset.html), master is the branch name of your remote origin.

Normally this error happens when you are not in local master branch when push and git can not solve which master you meant.   

## Create a local branch and force push to a remote branch <a name="forceLocalbranchtoRemotebranch"></a>
First, you create your branch locally:   

``` 
git checkout -b <branch-name>  
```

The remote branch is automatically created when you push it to the remote server. So when you feel ready for it, you can just do:  

```  
git push <remote-name> <branch-name>  
```

Where <remote-name> is typically origin, the name which git gives to the remote you cloned from. Your colleagues would then just pull that branch, and it's automatically created locally.  
  
Note however that formally, the format is:    

```  
git push <remote-name> <local-branch-name>:<remote-branch-name>  

# In case you have to force local branch to overwrite remote because remote branch already exists
git push -f <remote-name> <local-branch-name>:<remote-branch-name> 

# or using below syntax. + instead of -f, also for safety reason
git push <remote-name> +<local-branch-name>:<remote-branch-name>

# Force pushing more safely with --force-with-lease
git push <remote> <branch> --force-with-lease

# To delete a branch on the remote side, push an "empty branch" like so:
git push origin :deleteMe 
```
[notation +](http://stackoverflow.com/questions/39389380/how-to-overwrite-remote-branch-with-different-local-branch-with-git)

But when you omit one, *it assumes both branch names are the same*. Having said this, as a word of caution, do not make the critical mistake of specifying only :<remote-branch-name> (with the colon), or the remote branch will be deleted!  
  
So that *a subsequent git pull will know what to do*, you might instead want to use:  
  
git push -u <remote-name> <local-branch-name>  
As described below, the -u option sets up an upstream branch:  
  
For every branch that is up to date or successfully pushed, add upstream (tracking) reference, used by argument-less git-pull(1) and other commands.

Force pushing more safely with --force-with-lease

Force pushing with a "lease" allows the force push to fail if there are new commits on the remote that you didn't expect (technically, if you haven't fetched them into your remote-tracking branch yet), which is useful if you don't want to accidentally overwrite someone else's commits that you didn't even know about yet, and you just want to overwrite your own: [details](http://stackoverflow.com/questions/10510462/force-git-push-to-overwrite-remote-files)


## [Branch create and merge]<a name="BranchMerge"></a>
`$ git checkout -b amend-my-name` http://dont-be-afraid-to-commit.readthedocs.io/en/latest/git/commandlinegit.html#create-a-new-branch  
You can also choose what to base the new branch on. A quite common thing to do is, just for example:
`git checkout -b new-branch existing-branch`  

Master merge into Branch and solve all conflicts then merge Branch back into master. Safe practice.  
https://stackoverflow.com/questions/14168677/merge-development-branch-with-master   
```
(on branch development)$ git merge master
(resolve any merge conflicts if there are any)
git checkout master
git merge development (there won't be any conflicts now)
```  

## Checkout a remote branch <a name="remotebranch"></a>
[link](http://stackoverflow.com/questions/9537392/git-fetch-remote-branch)

```
git checkout -b serverfix origin/serverfix
```  
This is a common enough operation that git provides the --track shorthand:  
```
git checkout --track origin/serverfix
```  
In fact, this is so common that there’s even a shortcut for that shortcut. If the branch name you’re trying to checkout (a) doesn’t exist and (b) exactly matches a name on only one remote, Git will create a tracking branch for you:  
```
git checkout serverfix
```  
To set up a local branch with a different name than the remote branch, you can easily use the first version with a different local branch name:  
```
git checkout -b sf origin/serverfix
```  
Now, your local branch sf will automatically pull from origin/serverfix.  
  
Source: Pro Git, written by Scott Chacon and Ben Straub (cut for readability)  


## git diff merge tool <a name="diffmergetool"></a>

[Beyond compare 4 with git for windows](https://www.scootersoftware.com/support.php?zz=kb_vcs)
Use bc3 or Beyond compare 4
Path of BComp.exe depends on your Windows version

In case you like to use SourceTree, there's already a GUI config you can link it to the git merge, diff.

```
git config --global diff.tool bc3
git config --global difftool.bc3.path "C:\Program Files\Beyond Compare 4/BComp.exe"

git config --global merge.tool bc3
git config --global mergetool.bc3.cmd "C:\Program Files\Beyond Compare 4/BComp.exe"
```

when you need to use the tool:

```
git mergetool --tool=bc3
git mergetool --tool=sourcetree
```

And you need to solve the issues one by one manually [another tutorial](https://gist.github.com/karenyyng/f19ff75c60f18b4b8149)


## git submodules <a name=gitsubmodules></a> 
You want to use codes in another repo in your main project. You can create normally your own project and add other modules as submodules, here is [the steps explained](https://stackoverflow.com/questions/2140985/how-to-set-up-a-git-project-to-use-an-external-repo-submodule) 
```
cd MyWebApp
git submodule add git://github.com/jquery/jquery.git externals/jquery
```   
This will create a directory named externals/jquery* and link it to the github jquery repository. Now we just need to init the submodule and clone the code to it:
```
git submodule update --init --recursive
```


## git merge master update into branch <a name=mergeMasterBranch></a>

```
# https://stackoverflow.com/questions/16955980/git-merge-master-into-feature-branch
# do not risk and use git rebase, keep use git merge

git checkout feature1
git merge master

# safer option but create other chaos in later management:
# based on new master create another branch dev and merge feature branch into dev.
git checkout master
git branch dev

git checkout dev
git merge feature1
```


