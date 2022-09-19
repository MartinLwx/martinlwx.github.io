# 怎么在 homebrew 里面安装本地安装包


## 引言

---

最近想要在 homebrew 上面下载 qbittorent, 发现我即使用的是中科大的源也下载不下来, 终端显示如下内容⬇️

```bash
==> Downloading https://downloads.sourceforge.net/qbittorrent/qbittorrent-mac/qbittorrent-4.3.9/qbittorrent-4.3.9.dmg
curl: (35) error:06FFF089:digital envelope routines:CRYPTO_internal:bad key length=#   #   #

Error: Download failed on Cask 'qbittorrent' with message: Download failed: https://downloads.sourceforge.net/qbittorrent/qbittorrent-mac/qbittorrent-4.3.9/qbittorrent-4.3.9.dmg
```

此时我就想到要不干脆把这个文件下载下来, 然后用 homebrew 本地安装(应该是有这个功能的), 做了一番检索之后, 终于知道要怎么弄了, 下面我将以 qbittorrent-4.3.9.dmg 为例

## Step 1. 获取路径文件名

---

可以先运行 `brew --cache` 查看 homebrew 的缓存路径, 一般来说应该是在 `~/Library/Caches/Homebrew` 下.

homebrew 会把安装包下载到里面的 `downloads` 文件夹里面, 也就是在 `~/Library/Caches/Homebrew/downloads` 下, 进入这个文件夹可以发现里面的文件名的格式都是 `<url-hash>--<formula>-<version>`, 显然, 我们也要把我们的安装包弄成这种格式放在里面. 

> 📒 使用 `brew --cache -s <formula>` 来获取对应的路径文件名

对应我们这篇文章的例子就是 `brew --cache -s qbittorrent`, 可以看到输出内容是 `/Users/<对应你的用户名>/Library/Caches/Homebrew/downloads/7ee479ba2a19cf904e4c415805a6adaead76e7c191d595c016c86b72044c22fa--qbittorrent-4.3.9.dmg`

## Step 2. 移动本地安装包到对应的目录下

---

在 Step 1. 中我们已经可以知道该把文件放到什么地方, 接下来要做的无非就是移动文件可以, 如果打开文件浏览器移动那就慢了, 直接在命令行输入对应命令即可

> 📒 使用 `mv <local-file> "$(brew --cache -s <formula>)"` 移动本地安装包到对应目录下

对应我们的例子就是 `mv qbittorrent-4.3.9.dmg  "$(brew --cache -s qbittorrent)"`



## Step 3. 再次运行 `brew install`

---

此时再次运行 `brew install qbittorrent` 即可, 可以看到命令行找到了缓存文件:hugs:, 而且安装成功了

> 📒 再次运行 `brew install <formula>`

```bash
==> Downloading https://downloads.sourceforge.net/qbittorrent/qbittorrent-mac/qbittorrent-4.3.9/qbittorrent-4.3.9.dmg
Already downloaded: /Users/<对应你的用户名>/Library/Caches/Homebrew/downloads/7ee479ba2a19cf904e4c415805a6adaead76e7c191d595c016c86b72044c22fa--qbittorrent-4.3.9.dmg
==> Installing Cask qbittorrent
==> Moving App 'qbittorrent.app' to '/Applications/qBittorrent.app'
🍺  qbittorrent was successfully installed!
```

## 参考

---

1. [homebrew 文档](https://docs.brew.sh/Tips-N'-Tricks)


