# Git bundle guide


## What is the git bundle command
`git bundle` is a relatively less commonly used git command. Its purpose is to **package a git repo into a single file**, which can then be used by others to recreate the original git repo. Additionally, `git bundle` supports **incremental update**. Before I learned about the `git bundle` command, I would usually directly use `tar czf some_git_repo` to create a package for a git repo. Recently, I accidentally discovered the `git bundle` and found it quite useful🍻.

To better explain this command, let's use the folders `HostA` and `HostB` as an example, simulating two hosts. Let's assume that there is a git repo named `foo` on `HostA`, with the following directory structure

```
├── HostA
│   └── foo
│       ├── 1.txt
│       ├── 2.txt
│       └── 3.txt
├── HostB
```

There are 3 dummy commits in `foo`
```
* 21486d5 (HEAD -> main) add 3.txt
* a051186 add 2.txt
* 2820a6c add 1.txt
```
## Usage
### Full backup on HostA

For the first packaging, it is necessary to package the entire git repo. The command to package the `foo` repo on `HostA` is very simple
```sh
# in HostA/foo
# syntax: git bundle create <filename> <git-rev-list-args>
$ git bundle create foo.bundle HEAD main
```
It may be confusing what is the meaning of `<git-rev-list-args>`. You may regard it as **what we want to package into this bundle file**. *Here, we want to package the `foo` repo on `HostA`, which has a `main` branch. We also want to include the current location pointed to by `HEAD`, so we use `HEAD main`*

### Make a copy on HostB

Now, assuming that `HostB` has obtained the previously packaged `foo.bundle` file, recreating the original repo is also very simple, as shown in the following command
```sh
# in HostB
# syntax: git clone <filename> <target_dir>
$ git clone foo.bundle foo
```
As you can see, extracting information from the bundle file is similar to cloning a regular repository from a URL, except that we replace the URL with the path to the bundle file


At this point, if you check the remote repo info of this `foo` repo, you will notice that its remote repository has been set to the `foo.bundle` file
```sh
# in HostB/foo
$ git remote -v
# output:
# origin	<path_to_foo.bundle> (fetch)
# origin	<path_to_foo.bundle> (push)
```

### Making more commits on HostA
Now assuming that we have made more commits in `foo` repo on `HostA`
```
* 9ac69b0 (HEAD -> main) add 5.txt        -- new commit
* 0350a1e add 4.txt                       -- new commit
* 21486d5 add 3.txt
* a051186 add 2.txt
* 2820a6c add 1.txt
```

We want to package these two new commits and send the packaged bundle file to `HostB` so that we can make them in sync. By exploiting the syntax of specifying commit range[^1], we can use the following command to bundle the changes:
```sh
# in HostA/foo
# let's verify what will be bundled first
$ git log --oneline 21486d5..main

$ git bundle create increment.bundle 21486d5..main
```

### Incremental update on HostB

Again, let's assume that the `HostB` has obtained the `incremental.bundle` file. The following command can be used to extract the commits inside
```sh
# in HostB/foo
# syntax: git fetch <BUNDLE_FILE> <BRANCH_IN_BUNDLE>:<BRANCH_IN_LOCAL_REPO>
$ git fetch increment.bundle main:feature
```

The command above will dump the commits contained in the `incremental.bundle` to **a new `feature` branch**

```sh
# in HostB/foo
$ git log --oneline --graph --all
# output:
# * 9ac69b0 (feature) add 5.txt
# * 0350a1e add 4.txt
# * 21486d5 (HEAD -> main, origin/main, origin/HEAD) add 3.txt
# * a051186 add 2.txt
# * 2820a6c add 1.txt
```
**Once we are sure that there are no problems, we can attempt to merge the `feature` branch and delete it afterward**

```sh
# in HostB/foo
$ git merge feature
$ git branch -d feature
$ git log --oneline --graph --all
# output:
# * 9ac69b0 (HEAD -> main) add 5.txt
# * 0350a1e add 4.txt
# * 21486d5 (origin/main, origin/HEAD) add 3.txt
# * a051186 add 2.txt
# * 2820a6c add 1.txt
```

🤔️ You may wonder - Can we directly merge the commits from `incremental.bundle` into the `main` branch of `foo` repo on `HostB`? It's possible, with the command: `git pull increment.bundle main:main`. **However, it is not recommended to do so because `foo` repo on `HostB` may have also been updated. It is a good practice to first `fetch` and then `merge`**

You may still remember of that the remote branch of the `foo` repo on `HostB` was set to a specific bundle file. Can we directly use `git pull` to update `foo`? The answer is yes, we just need to rename the `increment.bundle` file to `foo.bundle` and place it in the path displayed by `git remote -v` (of course, changing the remote information is also an option)

## FAQ
How do we know what branches are included in a bundle file? The following command will handle this
```sh
# syntax: git bundle list-heads <BUNDLE_FILE>
$ git bundle list-heads  increment.bundle
# output: 9ac69b08060859bc4b2172a8238cb841846ec5e0 refs/heads/main
```

---

How do we know if we can use a bundle file in a specific repo? Move the bundle file into your git repo and use `git bundle verify`
```sh
# in HostB/foo
# syntax: git bundle verify <BUNDLE_FILE>
$ git bundle verify increment.bundle
# output:
# The bundle contains this ref:
# 9ac69b08060859bc4b2172a8238cb841846ec5e0 refs/heads/main
# The bundle requires this ref:
# 21486d53326de40678a54159de656714a59b8d09
# The bundle uses this hash algorithm: sha1
# increment.bundle is okay
```
*From the above output, we can see that the `increment.bundle` requires the repo to have the commit `21486d5` to be used for updating. The last commit of `foo` on `HostB` before synchronization is precisely this one*


## Wrap up

Using `git bundle` to package a git repository is indeed convenient. By combining it with the syntax for selecting a commit range, you can even choose only specific commits for incremental updates. This way, the bundle file will not be too large, making it easier for us to transfer

## Refs

[^1]: [Git Revision Selection](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)



