# Git Bundle 指南


## git bundle 是什么
`git bundle` 是一个比较少看到的 git 命令，**它的作用是把一个 git 仓库打包📦成一个文件，然后别人可以通过这个文件还原出本来的 git 仓库，而且 git bundle 还支持增量更新功能**。在知道 `git bundle` 命令之前，我有时候打包一个 git 仓库一般就直接 `tar czf some_git_repo`。前阵子偶然发现了 `git bundle` 发现还挺实用的🍻

这个命令最好通过例子来说明，*因此下面我用 `HostA` 和 `HostB` 这两个文件夹模拟两台主机，假设在 `HostA` 上存在某个 git 仓库 `foo`，目录结构如下*：

```
├── HostA
│   └── foo
│       ├── 1.txt
│       ├── 2.txt
│       └── 3.txt
├── HostB
```

其中 `foo` 有 3 个 commits，如下所示：
```
* 21486d5 (HEAD -> main) add 3.txt
* a051186 add 2.txt
* 2820a6c add 1.txt
```
## 使用场景
### HostA 上首次打包

首次打包那么必定是要对整个 git 仓库进行打包，打包 `HostA` 上的 `foo` 仓库的命令很简单
```sh
# in HostA/foo
# syntax: git bundle create <filename> <git-rev-list-args>
$ git bundle create foo.bundle HEAD main
```
比较难懂的是 `<git-rev-list-args>`，**它的含义是我们要把什么东西打包到这个 bundle 文件里面**。*这里我们是要打包 `HostA` 的 `foo` 仓库，它有一个 `main` 分支，同时我们把 `HEAD` 当前指向的位置也打包到这个 bundle 文件里面，所以用的是 `HEAD main`*

### HostB 上克隆仓库

现在假设 `HostB` 上拿到了前面打包好的 `foo.bundle` 文件，那么还原出本来的仓库也很简单，命令如下所示
```sh
# in HostB
# syntax: git clone <filename> <target_dir>
$ git clone foo.bundle foo
```

可以看到，提取 bundle 文件的信息就像是从一个普通的 URL 克隆仓库一样，只不过把 URL 换成了 bundle 文件的路径

此时看下这个仓库的远程仓库信息，你会发现它的远程仓库被设置成了 `foo.bundle` 文件
```sh
# in HostB/foo
$ git remote -v
# output:
# origin	<path_to_foo.bundle> (fetch)
# origin	<path_to_foo.bundle> (push)
```

### HostA 上继续更新
现在假设 `HostA` 上的 `foo` 仓库多更新了几个 commit，如下所示：
```
* 9ac69b0 (HEAD -> main) add 5.txt        -- new commit
* 0350a1e add 4.txt                       -- new commit
* 21486d5 add 3.txt
* a051186 add 2.txt
* 2820a6c add 1.txt
```
我们想要把 `HostA` 上这两个新的 commit 打包📦后发送给 `HostB` 让 `HostB` 可以和 `HostA` 保持同步。所以我们可以利用 git 提供的指定 commit range 的语法，选中这两个 commits[^1]
```sh
# in HostA/foo
# let's verify what will be bundled first
$ git log --oneline 21486d5..main

$ git bundle create increment.bundle 21486d5..main
```

### HostB 上增量更新

现在假设 `HostB` 拿到了 `increment.bundle`，那么用下面的命令可以提取里面的 commits
```sh
# in HostB/foo
# syntax: git fetch <BUNDLE_FILE> <BRANCH_IN_BUNDLE>:<BRANCH_IN_LOCAL_REPO>
$ git fetch increment.bundle main:feature
```
上面这个命令会将 `increment.bundle` 文件包含的 commit 放到**新的 `feature` 分支上**

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

**确定没有问题之后我们可以尝试合并 `feature` 分支，并删除 `feature` 分支**
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

🤔️ 那么能否直接将 `increment.bundle` 文件里的 commits 合并到 `HostB` 的 `foo` 的 `main` 分支上呢？也是可以的，命令是：`git pull increment.bundle main:main`，**但是并不建议这么做，因为 `HostB/foo` 也有可能更新了仓库，先 `fetch` 再 `merge` 是一个好习惯**

你可能还记得，`HostB` 上的 `foo` 这个仓库的远程分支被设置为某个文件，那么能否像一般使用 git 那样直接 `git pull` 呢？答案是肯定的，我们只需要将 `increment.bundle` 文件重命名为 `foo.bundle` 文件然后放在 `git remote -v` 显示的路径就行（当然改 remote 信息也可）
## FAQ
怎么知道 bundle 文件里面有什么分支可以用呢？下面的命令可以显示 bundle 文件里面所有的分支
```sh
# syntax: git bundle list-heads <BUNDLE_FILE>
$ git bundle list-heads  increment.bundle
# output: 9ac69b08060859bc4b2172a8238cb841846ec5e0 refs/heads/main
```

---

怎么确定一个 bundle 文件能否用于当前的某个 git 仓库上呢？只需要将 bundle 文件放在这个 git 仓库里然后运行 `git bundle verify`
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

*从上面的输出可以看到，`increment.bundle` 要求仓库有 `21486d5` 这个 commit 才能够被用来更新。前面的 `HostB/foo` 在同步之前的最后一个 commit 就是这个*
## 总结

`git bundle` 打包 git 仓库还是很方便的，结合选取 commit range 的语法还可以只选中部分 commits 增量更新，这样这个 bundle 文件就不会太大，方便我们进行传输

## 参考

[^1]: [Git Revision Selection](https://git-scm.com/book/en/v2/Git-Tools-Revision-Selection)



