# How to Manage Windows Using Hammerspoon


## Intro

---

I found that the windows management built in macOS is difficult to use. As a result, I always using my mouse to move and resize my window, which is less efficient. We should keep our hands on the keyboard as much as possible. After finishing the MIT-Missing-Semester, I came across the [hammerspoon](https://www.hammerspoon.org) tool. I really like this one :)

## What is hammerspoon ?

---

According to the official site's introduction, *hammerspoon is a tool for powerful automation of OS X, which is just a bridge between the operating system and a Lua scripting engine*. The windows management is kind of automation.



To be honest, I once heard the `Lua` language but I don't know anything about it. I follow [Learn Lua in Y minutes](https://learnxinyminutes.com/docs/lua/) to have a basic understanding of this language.

## How to manage windows ?

---

I would like to have the following features:

1.   Move and resize my window to the left/right of screen.
2.   The full screen mode
3.   Move and resize my window to the top-left/top-right/bottom-left/bottom-right of screen
4.   Move current window to the center of screen



My solution consists of 3 `*.lua` file(I have put this in my [dotfiles](https://github.com/MartinLwx/dotfiles))

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

This is the main part of windows management.

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

You shoule put these files in `~/.hammerspoon/` and then click on `Reload config` :hugs:

## Code Explained

---

-   `hs.hotkey.bind(mods, key, pressedfn)`
    -   This is a simple function to create a new hotkey and bind it to `pressedfn`.
    -   We press and hold `mods` and use `key` to enable `pressedfn`. For example, If we want to make a window full screen. We first press and hold `alt`(option) and `cmd`, then we press `f`.
-   `pressedfn`
    -   This is an anonymous function in `Lua`.
    -   The key of this function is `hs.window.focusedWindow():moveToUnit({...})`. Its job is get the *focused* window and make some changes of position and size. The parameters are a *table* in Lua. You may combine the previous comments in **window.lua** and the image below to understand this.

![](/img/movetounit.png)

## Ref

---

1.   [Anish’s Hammerspoon config](https://github.com/anishathalye/dotfiles-local/tree/mac/hammerspoon)

