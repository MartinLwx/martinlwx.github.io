# 怎么 debug 一个 Python 文件


## 引言

---

很长一段时间内我写代码都是用最简单的 debug 方法, 手动在程序里面插入 `print` 代码来看具体的变量的值, 然后自己推断程序到底是在哪里出问题. 根据 `print` 的结果可能还要到别的地方重复这个步骤. debug 完之后还得去把这些 `print` 语句注释掉. 这个就是我以前的日常 debug 流程.

最近在看 MIT.Missing semester 的课讲到 debug, 顿时感到应该系统学习一下在 Python 里面如何 debug, 虽然用 `print` 也凑合, 学完之后只恨自己没有早点了解:cry:

## 快速上手

---

**安装**

虽然 Python 有自带的 `pdb`, 但是 `ipdb` 的跟它大差不差, 还带颜色输出, 当然用这个了(其实就跟你在命令行要用 `python` 还是 `ipython` 一样, 我肯定是选择 UI 比较好看的 `ipython`)

```bash
> pip install ipdb
```

**开始 debug**

直接在命令行输入以下内容即可, 其中 `<filename>` 表示你要 debug 的文件

```bash
> python -m ipdb <filename>
```

## ipdb 中的命令

---

`[ ]` 表示是可选参数, 如果没有 `[ ]` 表示一定要给这个参数

`( )` 表示里面的可以不写, 简写命令

### **控制程序运行**

- `l(ist)` - 展示当前行附近的行, ~~具体来说是附近的 11 行, 但是记这个具体的数字好像也没啥意义~~
  - 也可以使用 `ll` 命令, 会展示当前所在函数或者是堆栈帧的源代码
  - 可以传入参数, 比如要看 1 到 12 行的内容可以用 `l 1,12`
- `s(tep)`  - 单步执行下一步, 如果是函数调用语句, 会进到函数里一步步执行
- `n(ext)` - 单步执行下一步, 如果是函数调用语句, 不会进去函数里一步步执行, 它会一直运行到函返回结果
- `c(ontinue)` \- 继续执行, 直到程序发生错误或者正常退出
  - 如果程序是正常退出的, 那么就会输出 `The program finished and will be restarted`
- `q(uit)` - 退出程序执行
- `r(eturn)` - 继续运行到当前函数返回结果为止
- `w(here)` - 打印 stack trace, 看调用轨迹, 从上到下分别是从最内层到最外层的调用入口
  - 然后你可以根据这个调用栈, 用 `d(own) [count]` 和 `u(up) [count]` 来在不同的层次间移动

### **程序断点专题**

每次设置程序断点的时候都会输出你当前这个程序断点的序号, 默认从 1 开始, 也就是后面提到的 `breakpoint count]`

- `b(reak) [line_number]` - 在 `line_number` 设置 breakpoint

  - 不提供参数的话就是查看我们设置的所有程序断点
  - :star: 高级用法, 你还可以指定文件! 比如你想停在 `util.py` 文件的第 10 行, 你可以用 `b util:10`
  - :star: 高级用法, 你在指定文件的同时, 可以指定函数, 还是刚才那个例子, 比如 `util.py` 文件里面有个 `get_result` 函数, 你可以用 `b util.get_result`
  - :star: 高级用法: 你还可以指定满足某些条件才会设置程序断点, 用法如下: `b ..., condition`

- `tbreak [line_number]` - 暂时的程序断点, 第一次命中之后就会自己取消

- `disable [breakpoint count]` - 暂时不用这个程序断点, 和 `clear` 不同, 你后面可以通过 `enable` 重新激活这个程序断点

- `enable [breakpoint count]` - 激活程序断点

- `cl(ear) [breakpoint count | line_number]` - 通过程序断点的序号或者是对应的行来清除程序断点

- `unt(il) [line_number]` - 运行大于等于 `line_number` 的地方

  - 当然你也可以不提供参数, 此时 `unt(il)` 命令会继续运行程序到行数比当前行大的那一行(有点绕口, 但意思其实就是下一行🧐), 此时它的功能类似于 `n(ext)`

    > 📒 `unt(il)` 命令默认会停在当前函数(或者是堆栈帧) return 的地方, 而 `c(ontinue)` 会一直运行到下去

- `restart [args...]` - 可以给定不同的参数再次重新运行来 debug

### **查看各种信息**

- `h(elp) [command]` - 不知道命令可以查询一下
- `p expression` - 相当于 `print expression`, 也可以使用 `pp`(对应 `pprint`)
  
  - 一个比较特殊的 `expression` 是 `locals()`, 可以查看当前所在位置的 context
  
    > 📒 要深刻理解这里是 `expression` 的好处, 如果本来的 `expression` 不对, 其实你可以直接在这里想要怎么改, 然后直接进行测试:open_mouth:
- `a(rgs)` - 打印当前所有的变量的值
- `whatis expression` - 相当于 `type(expression)`
- `source expression` - 查看 `expression` 的源码. 常用的 `expression` 是函数名

## 常见问题

---

Q: 我的变量名跟命令重复了怎么办, 比如变量名是 `p` ?

A: 这个其实不影响, 你还是可以通过 `p p` 来获取变量名的值. 如果你直接输入 `p` 显示 `p` 对应的值的话, 你可以使用 `!p`, 用 `!` 来告诉 `ipdb` 在它后面的是 python 语句



Q: 每次单步执行之后都要用 `p expression` 的方式来看变量的值, 有没有更为简便的方法?

A: 可以用 `display expression`, 那么在 `expression` 的值发生变动的时候, 它就会输出对应的值. 如果要取消就用 `undisplay expression`



Q: 觉得命令有点少不满足自己的需要 ?

A: 可以在 `ipdb` 里面直接写 python 的代码 :hugs:



Q: 不想使用 `python -m ipdb <filename>` 想直接在代码里插入程序断点 ?

A: 这也是可以的, 你可以在要插入的行之前设置, 像这样: 

```python
......
import ipdb; ipdb.set_trace();
......
```

如果你是用是 >= Python 3.7 的版本, 可以插入 `breakpoint()` 而不是 `import ipdb; ipdb.set_trace();`, 这样设置的好处是**你可以一次性取消所有的程序断点**:open_mouth:, 你就可以区分开正常运行的模式和调试程序的模式. 这种方式默认采用的是 `pdb`



Q: `->` 指向的行运行了吗 ?

A: 没有



Q: 如果我一直在调用 `step` 命令, 难道我每一次都要输入 `s` 吗, 有没有更快捷的方法 ?

A: 可以直接使用回车键(ENTER), 会自动重复上一次的命令

## 参考

---

1. [pdb 的官方文档](https://docs.python.org/3/library/pdb.html)

ps. 有兴趣的可以查看我翻译的一个项目 - [pdb 教程](https://github.com/MartinLwx/pdb-tutorial), 还挺有意思的

