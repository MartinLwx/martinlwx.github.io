# How to Manage Windows Using Hammerspoon


## 引言

---

虽然 macOS 自带窗口管理这个功能, 但是实际上是用下来发现还是很难受的. 功能不足以满足自己的需求. 所以我常常发现自己在用鼠标拖动窗口和重新调整窗口大小. 长此以往, 我觉得这样效率实在太低, 恰好前阵子在看 MIT-Missing-Semester 的课, 里面提到了 [hammerspoon](https://www.hammerspoon.org) 这个工具. 我去稍微了解了一下发现这工具真的不错.

## 什么是 hammerspoon ?

---

根据官方文档介绍, hammerspoon 是 macOS 上一个用于自动化的工具, 充当了 Lua 语言和操作系统的系统调用之间的桥梁. 也就是说我们可以使用 Lua 语言和 hammerspoon 提供的 API 来来完成很多自动化操作. 目前我还只看了窗口管理相关的.



因为要用到 Lua 语言, 但是我又根本没有接触过这个语言, 所以我大概跟着 [Learn Lua in Y minutes](https://learnxinyminutes.com/docs/lua/) 稍微学习了一下.

## 怎么管理窗口?

---

我主要想要有下面的几个功能

1.   可以把窗口移动到屏幕的左边或者是右边(1 / 2 屏幕)
2.   可以把窗口弄成全屏的
3.   可以把窗口移动到左上角/右上角/左下角/右下角(1 / 4 屏幕)
4.   把当前的窗口移动到屏幕中央



我的方案主要是写了三个 `*.lua` file(我放在了我的 github 仓库 [dotfiles](https://github.com/MartinLwx/dotfiles) 里)

:point_right:**config.lua**

```lua
MACBOOK_MONITOR = 'Built-in Retina Display'

-- disable animations, default value = 0.2
hs.window.animationDuration = 0
```

:point_right:**init.lua**

```lua
require('config')
require('window')
```

:point_right:**window.lua**

这个是主要的代码

```lua
-- half of screen
-- {frame.x, frame.y, window.w, window.h}
-- First two elements: we decide the position of frame
-- Last two elements: we decide the size of frame
hs.hotkey.bind({'alt', 'cmd'}, 'left', function() hs.window.focusedWindow():moveToUnit({0, 0, 0.5, 1}) end)
hs.hotkey.bind({'alt', 'cmd'}, 'right', function() hs.window.focusedWindow():moveToUnit({0.5, 0, 0.5, 1}) end)
hs.hotkey.bind({'alt', 'cmd'}, 'up', function() hs.window.focusedWindow():moveToUnit({0, 0, 1, 0.5}) end)
hs.hotkey.bind({'alt', 'cmd'}, 'down', function() hs.window.focusedWindow():moveToUnit({0, 0.5, 1, 0.5}) end)

-- quarter of screen
--[[
    u i
    j k
--]]
hs.hotkey.bind({'ctrl', 'alt', 'cmd'}, 'u', function() hs.window.focusedWindow():moveToUnit({0, 0, 0.5, 0.5}) end)
hs.hotkey.bind({'ctrl', 'alt', 'cmd'}, 'k', function() hs.window.focusedWindow():moveToUnit({0.5, 0.5, 0.5, 0.5}) end)
hs.hotkey.bind({'ctrl', 'alt', 'cmd'}, 'i', function() hs.window.focusedWindow():moveToUnit({0.5, 0, 0.5, 0.5}) end)
hs.hotkey.bind({'ctrl', 'alt', 'cmd'}, 'j', function() hs.window.focusedWindow():moveToUnit({0, 0.5, 0.5, 0.5}) end)


-- full screen
hs.hotkey.bind({'alt', 'cmd'}, 'f', function() hs.window.focusedWindow():moveToUnit({0, 0, 1, 1}) end)

-- center screen
hs.hotkey.bind({'alt', 'cmd'}, 'c', function() hs.window.focusedWindow():centerOnScreen() end)
```

你应该把上面三个文件放在 `~/.hammerspoon/` 这个路径下然后在 hammerspoon 里面点击 `Reload config` 就可以正常使用了:hugs:

## 代码解释

---

-   `hs.hotkey.bind(mods, key, pressedfn)`
    -   这个主要是把按键 `mods` 和 `key` 绑定到 `pressedfn` 这个函数上.
    -   在使用的时候先按下 `mods` 对应的按键组合并保持, 然后再按下 `key` 对应的按键. 比如我们要让窗口全屏, 我们首先按下并保持 `alt`(option) 和 `cmd`, 再按下 `f`. 就可以让窗口全屏了
-   `pressedfn`
    -   这个就是用 `Lua` 语言写的一个函数
    -   函数的关键在于 `hs.window.focusedWindow():moveToUnit({...})` 这个方法. 他的主要功能是获得当前被激活的窗口然后做一些位置和大小上的修改. 参数是 Lua 语言里的 table, 我已经把参数的具体含义写在了注释里, 可以结合下面我画的图来进行理解.



## 参考

---

1.   [Anish’s Hammerspoon config](https://github.com/anishathalye/dotfiles-local/tree/mac/hammerspoon)

