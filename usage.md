# Using submodules in Git

## [1. Submodules - repositories inside other Git repositories](https://www.vogella.com/tutorials/GitSubmodules/article.html#submodules)

### [1.1. Using Git repositories inside other Git repositories](https://www.vogella.com/tutorials/GitSubmodules/article.html#using-git-repositories-inside-other-git-repositories)

Git allows you to include other Git repositories called _submodules_ into a repository. This allows you to track changes in several repositories via a central one. Submodules are Git repositories nested inside a parent Git repository at a specific path in the parent repository’s working directory. A submodule can be located anywhere in a parent Git repository’s working directory and is configured via a `.gitmodules` file located at the root of the parent repository. This file contains which paths are submodules and what URL should be used when cloning and fetching for that submodule. Submodule support includes support for adding, updating, synchronizing, and cloning submodules.

Git allows you to commit, pull and push to these repositories independently.

Submodules allow you to keep projects in separate repositories but still be able to reference them as folders in the working directory of other repositories.

## [2. Working with repositories that contain submodules](https://www.vogella.com/tutorials/GitSubmodules/article.html#working-with-repositories-that-contain-submodules)

### [2.1. Cloning a repository that contains submodules](https://www.vogella.com/tutorials/GitSubmodules/article.html#cloning-a-repository-that-contains-submodules)

If you want to clone a repository including its submodules you can use the `--recursive` parameter.

```
git clone --recursive [URL to Git repo]
```

If you already have cloned a repository and now want to load it’s submodules you have to use `submodule update`.

```
git submodule update --init
# if there are nested submodules:
git submodule update --init --recursive
```

### [2.2. Downloading multiple submodules at once](https://www.vogella.com/tutorials/GitSubmodules/article.html#downloading-multiple-submodules-at-once)

Since a repository can include many submodules, downloading them all sequentially can take much time. For this reason `clone` and `submodule update` command support the `--jobs` parameter to fetch multiple submodules at the same time.

```
# download up to 8 submodules at once
git submodule update --init --recursive --jobs 8
git clone --recursive --jobs 8 [URL to Git repo]
# short version
git submodule update --init --recursive -j 8
```

### [2.3. Pulling with submodules](https://www.vogella.com/tutorials/GitSubmodules/article.html#submodules_pulling)

Once you have set up the submodules you can update the repository with fetch/pull like you would normally do. To pull everything including the submodules, use the `--recurse-submodules` and the `--remote` parameter in the `git pull command`.

```
# pull all changes in the repo including changes in the submodules
git pull --recurse-submodules

# pull all changes for the submodules
git submodule update --remote
```

### [2.4. Executing a command on every submodule](https://www.vogella.com/tutorials/GitSubmodules/article.html#executing-a-command-on-every-submodule)

Git provides a command that lets us execute an arbitrary shell command on every submodule. To allow execution in nested subprojects the `--recursive` parameter is supported. For our example we assume that we want to reset all submodules.

```
git submodule foreach 'git reset --hard'
# including nested submodules
git submodule foreach --recursive 'git reset --hard'
```

## [3. Creating repositories with submodules](https://www.vogella.com/tutorials/GitSubmodules/article.html#creating-repositories-with-submodules)

### [3.1. Adding a submodule to a Git repository and tracking a branch](https://www.vogella.com/tutorials/GitSubmodules/article.html#submodules_trackbranch)

If you add a submodule, you can specify which branch should be tracked via the `-b` parameter of the `submodule add` command. The `git submodule init` command creates the local configuration file for the submodules, if this configuration does not exist.

```
# add submodule and define the master branch as the one you want to track
git submodule add -b master [URL to Git repo]

# initialize submodule configuration
git submodule init
```

If you track branches in your submodules, you can update them via the `--remote` parameter of the `git submodule update` command. This pulls in new commits into the main repository and its submodules. It also changes the working directories of the submodules to the commit of the tracked branch.

```
# update your submodule --remote fetches new commits in the submodules
# and updates the working tree to the commit described by the branch
git submodule update --remote
```

### [3.2. Adding a submodule and tracking commits](https://www.vogella.com/tutorials/GitSubmodules/article.html#submodules_adding)

Alternatively to the tracking of a branch, you can also control which commit of the submodule should be used. In this case the Git parent repository tracks the commit that should be checked out in each configured submodule. Performing a submodule update checks out that specific revision in the submodule’s Git repository. You commonly perform this task after you pull a change in the parent repository that updates the revision checked out in the submodule. You would then fetch the latest changes in the submodule’s Git repository and perform a submodule update to check out the current revision referenced in the parent repository. Performing a submodule update is also useful when you want to restore your submodule’s repository to the current commit tracked by the parent repository. This is common when you are experimenting with different checked out branches or tags in the submodule and you want to restore it back to the commit tracked by the parent repository. You can also change the commit that is checked out in each submodule by performing a checkout in the submodule repository and then committing the change in the parent repository.

You add a submodule to a Git repository via the `git submodule add` command.

```
# add a submodule to the existing Git repository
git submodule add [URL to Git repo]

# initialize submodule configuration
git submodule init
```

### [3.3. Updating which commit your are tracking](https://www.vogella.com/tutorials/GitSubmodules/article.html#submodules_track)

The relevant state for the submodules are defined by the main repository. If you commit in your main repository, the state of the submodule is also defined by this commit.

The `git submodule update` command sets the Git repository of the submodule to that particular commit. The submodule repository tracks its own content which is nested into the main repository. The main repository refers to a commit of the nested submodule repository.

Use the `git submodule update` command to set the submodules to the commit specified by the main repository. This means that if you pull in new changes into the submodules, you need to create a new commit in your main repository in order to track the updates of the nested submodules.

The following example shows how to update a submodule to its latest commit in its master branch.

```
# update submodule in the master branch
# skip this if you use --recurse-submodules
# and have the master branch checked out
cd [submodule directory]
git checkout master
git pull

# commit the change in main repo
# to use the latest commit in master of the submodule
cd ..
git add [submodule directory]
git commit -m "move submodule to latest commit in master"

# share your changes
git push
```

Another developer can get the update by pulling in the changes and running the submodules update command.

```
# another developer wants to get the changes
git pull

# this updates the submodule to the latest
# commit in master as set in the last example
git submodule update
```

**Warning**

> _With this setup you need to create a new commit in the master repository, to use a new state in the submodule. You need to repeat this procedure every time you want to use another state in one of the submodules. See[Adding a submodule to a Git repository and tracking a branch](https://www.vogella.com/tutorials/GitSubmodules/article.html#submodules_trackbranch) for tracking a certain branch of a submodule._

### [3.4. Delete a submodule from a repository](https://www.vogella.com/tutorials/GitSubmodules/article.html#delete-a-submodule-from-a-repository)

Currently Git provides no standard interface to delete a submodule. To remove a submodule called `mymodule` you need to:

- git submodule deinit -f — mymodule
- rm -rf .git/modules/mymodule
- git rm -f mymodule
