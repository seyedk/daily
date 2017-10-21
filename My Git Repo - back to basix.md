This is how I'm gonna keep my memory refreshed when I'm working with git on command line interface. 
## Create a new Remote Repository on github

### 1) More generic way, using the github API version 3.0+
```
curl -u 'USER' https://api.github.com/user/repos -d '{"name":"REPO"}'
# Remember replace USER with your username and REPO with your repository/application name!
git remote add origin git@github.com:USER/REPO.git
git push origin master
```

### 2) github CLI: Add your code to a new repository 
When you already have a repo and need to push a brand new code to it, you can follow this instructions (which is a copy / paste from the github new repo page:)

```
echo "# pyExpress" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/seyedk/pyExpress.git
git push -u origin master
```

### if you already have a repo, then push an existing repository from the command line
```
git remote add origin https://github.com/seyedk/pyExpress.git
git push -u origin master
```

### 



Now let's ***clone*** a `remote` repo (master) or creating a new `local` repo:

`+` Cloning a git repo. I like this one a lot:
```
git clone https://github.com/seyedk/aws-discovery-utils.git
```
`+` Initiating a repo off of the current directory:

``` cd ~/myproject
git init
```
after creating new files for your project, you need to add them to the repor

```
git add mygame.py
git add LICENSE
git commit -m 'initial project version'

```

## Other Useful Git commands 
### Check the status
```
git status
```
### Short Status
```

git status -s 

```

the result will be something like this:

```
M README
MM test.py
A  tests/uat.py
M  lib/games.py
```
A: means added
M: means it's modified 
MM: Modified, Staged, and then modified again.
??: file has not been tracked.

### Committing the changes:
use one of the followings
``` 
 git commit
```

```
 git commit -m 'your comment goes here'
```

## basic branching / merging 
when I need to develop a new feature or work on an issue, I create a local of my main branch (such as my master, or dev, or integration branches).
in this case, first decide the name of the branch, which let's say it's my work on issue 4004. This may go live independent to any hotfix that may happen meanwhile. let's go with basix first:
```
git checkout -b issue4004
```
This simply combines to commnands of "create branch" and then checkout:

```
git branch issue4004
git checkhout issue4004
```

then after working ***hard*** on the issue, I just add my new files and commit them. sometimes I just go ahead and combine the merge and add commands in one:
```
git commit -a -m 'the hard work: issue 4004 is done 
```

now after rounds of test, it comes to merging this branch to my master. here's how I will do it; first make my main (master) branch the active branch 
```
git checkout master
```
Then merge the branch to it
```
git merge issue4004
```

if no longer needed, I will go ahead delete the issue4004 branch

```
git branch -d issue4004
```