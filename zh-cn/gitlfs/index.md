# git lfs 使用指南


## 什么是 `git-lfs`

---

### Github 对文件大小的限制

如果你在命令行用 `git push`  > 50 MB 的文件，你会收到一个 `warning`，但是你仍然可以正常 `push`，但是 > 100 MB 的时候就无法 `push` 了

如果你在浏览器要上传文件的话，限制更为严重，不能超过 25 MB

另外有几点值得注意：

1. Github 建议仓库的大小理想情况下不要超过 1 GB，最好不要超过 5 GB
2. Github 从来不建议把仓库当成一种备份工具

### 为什么需要 `gif-lfs` 

前面提到的 Github 对文件大小的限制是一点

另外因为每次我们在使用 `git commit` 的时候，其实是给当前的仓库创建了一次快照，本质是全仓库的克隆，如果大文件太多是很不好的，你的 Git 仓库会越来越大

### 什么情况下不需要用 `gif-lfs`

1. 文件没有超过限制当然就没有必要用了
2. 如果是要分发二进制文件（比如 *.exe）等，此时直接用 Github 提供的 release 功能就好了

## `git-lfs` 原理

---

使用 `gif-lfs` 之后，在仓库中存储的其实是对大文件的引用，可以理解为指针。而真正的大文件托管在 Git Lfs 的服务器上

Github 给不同用户的 `git-lfs` 提供的额度不一样，免费用户和 Pro 用户都是 2 GB

### 引用文件长什么样子

比如官方文档里面提到的例子：

```
version https://git-lfs.github.com/spec/v1
oid sha256:4cac19622fc3ada9c0fdeadb33f88f367b541f38b89102a3f1261ac81fd5bcb5
size 84977953
```

其中 `version` 是你正在使用的 `git-lfs` 的版本，`oid` 是标志符（id），`size` 是文件的真实大小

## 开始使用 `git-lfs`

---

### 如何安装 `git-lfs` (Mac 环境下)

```bash
> brew install git-lfs
> git lfa install                 # 如果输出为 Git LFS initialized. 就是正常安装好了
```

### Case 1. 从 0 开始配置使用 `git-lfs`

我们要指定 `git-lfs` 会把哪些文件当作大文件，指定方式比如有：

1. 指定文件后缀名——`git lfs track "*.filetype"`
2. 指定某个目录下的所有文件——`git lfs track "directory/*"`
3. 具体指定某个文件——`git lfs track "path/to/file"`

```bash
> mkdir <repo>
> cd <repo>
> git init
> git lfs track "*.filetype"			# 比如 *.zip

# 其实 git lfs track 会修改 .gitattributes 文件的内容，可以做一个快速的验证
# > cat .gitattributes
# *.zip filter=lfs diff=lfs merge=lfs -text

# 下面假定在 Github 有一个远程仓库供我们使用
# 往仓库里加你先前指定的文件类型的大文件
> git add . 											
> git commit -m ""
> git branch -M main
> git remote add origin git@github.com:<username>/<remote_repo_name>.git		# 这里替换为自己的用户名和远程仓库名
> git push -u origin main
# 此时命令行会显示
# > uploading LFS objects
# 如果没有采用 git-lfs，则显示如下内容
# > Enumerating objects: 3, done.
#   Counting objects: 100% (3/3), done.
#   Delta compression using up to 8 threads
#   Compressing objects: 100% (2/2), done.
```

### Case 2. 要在已有的仓库上用 `git-lfs` 追踪某些文件

此时只是简单的使用 `git lfs track ""` 是没用的，因为你之前的 commit 已经生成了快照，你无法追踪历史中的这些大文件。

**`git-lfs` 只会在你开始设置的此刻之后追踪新生成的指定文件**

可以快速做个验证，假设我们还在这个仓库里⬇️

```bash
> ls > test1.txt
> ls -l > test2.txt
> git add test1.txt test2.txt
> git commit -m "Add txt files"

# 假设我们现在要把 txt 文件当成是大文件，我们可能会想这么做
> git lfs track "*.txt"
> git add .gitattributes
> git commit -m "Track *.txt files"
> git lfs ls-files                      # 此时你会发现 git-lfs 并没有追踪 txt 文件

> echo "hello" > test3.txt
> git add test3.txt
> git commit -m "Add test3.txt"
> git lfs ls-files                      # 此时你可以在输出中看到 test3.txt
```

正确的方法是使用 `git lfs migrate`，这里只列举了简单的用法，更复杂的可以看看手册。比如可以用 `--include-ref=` 指定分支，多个分支的时候最好一个分支一个分支地迁移，最后是 `git push --all -f`

```bash
> git lfs migrate import --include="*.txt"  # 在当前分支上执行
> git lfs ls-files                          # 此时可以发现 text1.txt 和 text2.txt 也被追踪到了
> git push --force                          # 让远程仓库也改过来
```

### Case 3. 不再跟踪某些文件

```bash
> git lfs untrack "*.filetype"
> git rm --cached "*.txt"
```

## 其他常用命令

---

1. 查看当前 `git-lfs` 正在追踪的文件类型——`git lfs track`

2. 查看当前 `git-lfs` 正在追踪哪些文件——`git lfs ls-file`

## 参考

---

1. https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github

