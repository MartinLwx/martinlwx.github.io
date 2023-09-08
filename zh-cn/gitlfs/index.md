# Git LFS 使用指南


## 为什么需要 Git LFS 

如果你在命令行用 `git push` > 50MB 的文件，你会收到一个 `warning`，但是你仍然可以正常 `push`，但是 > 100MB 的时候就无法 `push` 了。如果你是在浏览器要上传文件的话，这个限制更为严重，不能超过 25MB，这是 Github 对仓库的限制。Git lfs 就是用于解决这个问题[^1]


### 什么情况下不需要用 Git LFS

下面几个场景不需要用
1. 文件**大小没有超过限制**当然就没有必要用了
2. 如果是要**分发二进制文件（比如 `*.exe`）等**，此时直接用 Github 提供的 release 功能就好了

## Git LFS 原理


使用 Git LFS 之后，在仓库中存储的其实是对大文件的引用，可以理解为指针。而真正的大文件托管在 Git Lfs 的服务器上。Github 给不同用户的提供的存储空间不一样，免费用户和 Pro 用户都是 2 GB，而如果是企业用户则会高点[^2]

### 引用文件长什么样子

比如官方文档里面提到的例子：

```
version https://git-lfs.github.com/spec/v1
oid sha256:4cac19622fc3ada9c0fdeadb33f88f367b541f38b89102a3f1261ac81fd5bcb5
size 84977953
```

其中 `version` 是你正在使用的 `git-lfs` 的版本，`oid` 是标志符，`size` 是文件的实际大小

## 开始使用 Git LFS


### 如何安装 Git LFS

如果用的是 Mac，那么用 Homebrew 就可以很方便安装

```sh
> $ brew install git-lfs
> $ git lfa install                 # 如果输出为 Git LFS initialized. 就是正常安装好了
```

### Case 1. 从 0 开始配置使用 Git LFS

我们要指定 Git LFS 会把哪些文件当作大文件，指定方式比如有：

1. 指定文件后缀名——`git lfs track "*.filetype"`
2. 指定某个目录下的所有文件——`git lfs track "directory/*"`
3. 具体指定某个文件——`git lfs track "path/to/file"`

```sh
> $ mkdir <repo>
> $ cd <repo>
> $ git init
> $ git lfs track "*.filetype"			# 比如 *.zip

# git lfs track 会修改 .gitattributes 文件的内容，可以验证一下
# > cat .gitattributes
# *.zip filter=lfs diff=lfs merge=lfs -text

# 下面假定在 Github 有一个远程仓库供我们使用
# 往仓库里加你先前指定的文件类型的大文件
> $ git add .
> $ git commit -m "<msg>"
> $ git branch -M main
# 这里替换为自己的用户名和远程仓库名
> $ git remote add origin git@github.com:<username>/<remote_repo_name>.github 
> $ git push -u origin main

# 此时命令行会显示
# > uploading LFS objects

# 如果没有采用 git-lfs，则显示如下内容
# > Enumerating objects: 3, done.
#   Counting objects: 100% (3/3), done.
#   Delta compression using up to 8 threads
#   Compressing objects: 100% (2/2), done.
```

### Case 2. 要在已有的仓库上用 Git LFS 追踪某些文件

此时只是简单的使用 `git lfs track ...` 是没用的，因为你之前的 commit 已经生成了快照，你无法追踪历史中的这些大文件。**Git LFS 只会在你开始设置的此刻之后追踪新生成的指定文件**

可以快速做个验证，假设我们还在这个仓库里⬇️

```bash
> $ ls > test1.txt
> $ ls -l > test2.txt
> $ git add test1.txt test2.txt
> $ git commit -m "Add txt files"

# 假设我们现在要把 txt 文件当成是大文件，我们可能会想这么做
> $ git lfs track "*.txt"
> $ git add .gitattributes
> $ git commit -m "Track *.txt files"
> $ git lfs ls-files                      # 此时你会发现 git-lfs 并没有追踪 txt 文件

> $ echo "hello" > test3.txt
> $ git add test3.txt
> $ git commit -m "Add test3.txt"
> $ git lfs ls-files                      # 而用 LFS 追踪新文件是没问题的
```

正确的方法是使用 `git lfs migrate`，这里只列举了简单的用法，更复杂的可以看看手册
```python
> $ man git-lfs-migrate
```

*比如可以用 `--include=`，Git LFS 会自动根据我们指定的模式进行选择。下面的 `--include="*.txt` 的意思就是选中之前的 txt 文件*

```bash
> git lfs migrate import --include="*.txt"

# 此时可以发现 text1.txt 和 text2.txt 也被追踪到了
> git lfs ls-files

# 让远程仓库也改过来
> git push --force
```

### Case 3. 不再跟踪某些文件

```bash
> git lfs untrack "*.filetype"
> git rm --cached "*.filetype"
```

## 其他常用命令


1. 查看当前 Git LFS 正在追踪的文件类型——`git lfs track`

2. 查看当前 Git LFS 正在追踪哪些文件——`git lfs ls-file`

## 参考


[^1]: [About large files on GitHub](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github)
[^2]: [About Git Large File Storage](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage)

