# Vim Macro 101


## Intro

In the process of learning Vim, the most enlightening sentence for me is : 

> **Vim is a kind of *programming language***

I once tried to learn Vim a long time ago. However, the keystroke combinations are very difficult to memorize in my opinion. So I later gave up learning Vim and switched to a normal text editor. My view of Vim changed when I took this course: *[Missing semester](https://missing.csail.mit.edu/)*. I started to regard Vim as a kind of programming language rather than just a text editor. Then I find that **the combinations of various operations of Vim are the syntax of programming language**.

It seems that I am talking about something unrelated to this blog post. That's not true. As programmers, we often avoid doing repetitive tasks by programming. Different people have different favorites. Vim is one of the options you probably do not know.

> I assume you have a basic understanding of basic Vim usage  : )

## What is macro in Vim?

We know that we can use `.` to repeat the last change. However, what if we want to apply many changes? Macro comes to the rescue. :)

By using macro, we can record our operations and then store them in a register which can be executed later. If you check the content of the corresponding register, you may find that it is just a sequence of characters.

## Macro-how?

### How to record a macro

**Steps to follow**:

1. In normal mode, press `q<register>`.
   1. It means you first press `q` and then choose a `<register>` you like to store your macro.
   2. The `regitser`s you can use are: `0-9`, `a-z`, and `"`.

2. Start typing. If everything goes right, you will see `recording @<register>` in the bottom left. From now on, everything you type will be recorded.

3. Press `q` again to quit the recording.


### How to inspect a macro
I previously mentioned that the macro is just a sequence of characters stored in a specific register. We can use the following command to inspect the content:

```sh
:reg <register>
```

You may see some strange characters:

- `^[` means `<ESC>`
- `<80>kb` means `BAKESPACE`

### How to execute a macro
#### Within a file

- `@<register>`: execute the macro in `<register>`
- `@@`: execute the last executed macro

What if we want to execute a macro many times? Well, it is easy. We just add `[COUNT]` before `@@` or `@<register>`. Remember, Vim is a kind of *programming language*. Do all these make sense if you consider it as a kind of programming language?
  - e.g. *`100@@` means executing the last executed macro 100 times*

Calculating what the value of `[COUNT]` should be is tedious. Luckily, we can set it to a large value. The exceeding part will be ignored.

#### A bunch of files
Things got complicated when you tried to modify multiple files. Opening each file and applying a macro repeatedly would be unreasonable. It is a violation of the DRY(Don't repeat yourself) principle.

Vim allows us to open multiple files at the same time and edit them one by one. What's more, we can apply our recorded macro to all files with a single command.

For example, you may want to edit all the `txt` files in the current directory. Typing `vim *.txt` in your terminal will open all these files. At first glance, it looks like you are editing a file, except that if you try to type `:x` to save and exit you will get an error! The naive way is modifying the file content and then typing `:wn` to save the changes and go to the next file. Now I will tell you how to do this more cleverly.

1. Record your macro first then drop the changes by typing `:edit!` in the 1-st file because we will change all files including the 1-st file later. Without this cancellation, the 1st file will be modified twice.
2. In normal mode, type `:bufdo execute "normal @<register>" | update`.

More details can be found [here](https://vim.fandom.com/wiki/Run_a_command_in_multiple_buffers)


### How to modify a macro

#### Simple case

If you are recording a quite long macro and you press `q` unexpectedly, you definitely do not want to start from the beginning. Vim has a feature to save us from this trouble.

In Vim, if we press `q<REIGSTER>`(`<REGISTER>` means `A-Z`) in normal mode. Everything we typed now will be appended to the corresponding `register`. *For example, we may have recorded macro in register `a`, then we can press `qA` to append operations to this marco*. That is how we continue recording a half-finished macro.

#### Complex case

What if we want to change the operations in the middle rather than appending them? The secret is that we can just treat the macro as a text sequence. After editing, we can store it back:

1. Press `G` to jump to the last line, then type `:put <register>`. The content of the `register` will appear in the next line.
2. Just edit this mysterious text sequence.
3. After you have edited the content, you can type `:d <register>` to put back the content(make sure your cursor is still in this line).

e.g. *The following gif shows the procedure of modifying a macro. I have a dumb macro in the register `y` which can append ` world` in the current line. I change it to append ` hello world` instead.*

![](/img/modify_macro.gif)


## Wrap up
This is a simple tutorial about the macro in Vim, which is just the tip of the iceberg in Vim. Only when you master Vim do you make the most of it. It would be great if this blog got you interested in learning Vim. Hope this tutorial will be helpful 😉

