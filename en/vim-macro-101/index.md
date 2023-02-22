# Vim Macro 101


## Intro

In the process of learning `vim`, the most enlightening sentence for me is : 

> **vim is a kind of programming language**

I once try to learn `vim` a long time ago. However, the key combination logic is very difficult to remember in my opinion. So I later gave up learning `vim` and switched to a normal text editor. My view of `vim` changed when I met this course: *[Missing semester](https://missing.csail.mit.edu/)*. I started to regard `vim` as a kind of programming language rather than just a text editor. :)

It seems that I am talking about something unrelated to this blog post. That's not true. As programmers, we often do repetitive tasks by programming. Different people have different favorites. `vim` is definitely one of the options for solving problems. 

## What is macro in vim?

> I assume you have a basic understanding of basic `vim` usage  : )

We know that we can use `.` to repeat the last change, well, a simple change I mean. Have you wondered if there is a more powerful way to repeat **complex changes**?


Yeah, It is called `macro` in `vim`. You can record your typed characters(operations) into a register and execute them later. It is just a sequence of characters that could be understood by `vim`.

## Macro-how?

### How to record your macro

**Steps followed**:

1. In normal mode, press `q<register>`.
   
   1. It means you first press `q` and then choose a `register` you like to store your macro.
   
   2. the `regitser`s you can use: `0-9`, `a-z`, and `"`. I will explain why not use the uppercase letters here later

2. Start typing. If everything goes right, you will see `recording @<register>` in the bottom left. From now on, everything you typed will be recorded.

3. Press `q` again to exit the recording.

> 📒 The most important thing when you record your macro is: you should **make all steps repeatable in other similar files**. So, the takeaway is: look into the files and figure out the common pattern before recording :)

### How to check my macro
I previously mentioned that the macro is just a sequence of characters stored in a specific register. So if we want to look into the register, we just type the `:reg` command.

```sh
:reg <register>
```

You may see some strange characters: 

- `^[` means `<ESC>`

- `<80>kb` means `BAKESPACE`

### How to execute my macro
#### Within a file

- `@<register>`: execute the macro in `<register>`

- `@@`: execute the previously executed macro

- What if I want to execute my macro many times? Well, it is easy. You just add `[COUNT]` before `@@` or `@<register>`. Remember, `vim` is a kind of programming language. Do all these make sense if you consider it as a kind of programming language?
  - e.g. `100@@` means executing the previous executed macro 100 times : )

It is worth mentioning that if you want to apply your macro within a file, you can set a larger `[COUNT]`. The exceeding part will be ignored.
#### a bunch of files
Open a file and apply a macro repeatedly could be very tedious. It is absolutely a violation of the DRY(Don't repeat yourself) principle.

`vim` allows us to open multiple files at the same time and edit them one by one. What's more, we can apply our macro to all files in a single command.

For example, you want to edit all the `txt` files in the current directory. You can open all of them by typing `vim *.txt` in your terminal. At first glance, it looks like you are editing a file, except that if you try to type `:x` to save and exit you will get an error! The basic way is modifying and then typing `:wn` to save the changes and go to the next file. Now I will tell you how to do this cleverly.

1. Record your specific macro first then drop the changes by typing `:edit!` in the 1st file because we will change all files later. Without this step, the 1st file will be modified twice.
2. Typing `:bufdo execute "normal @<register>" | update`.

More details can be found [here](https://vim.fandom.com/wiki/Run_a_command_in_multiple_buffers)


### How to modify my macro

#### simple case

If you are recording a quite long macro and you press `q` unexpectedly, you may need this feature. It is good news that we don't need to start from the beginning again.


Do you still remember that I deliberately skip the uppercase letter registers earlier? Actually. **When you press `q<REIGSTER>`(`REGISTER` means `A-Z`). Everything you typed will be appended to the `REGISTER`** 🚀

#### complex case

You may need to change some operations in the middle of the macro. Still, you don't need to redo all these.

You may follow the steps to modify your macro:

1. Press `G` to jump to the last line, then type `:put <register>`. You will see the content of the `register` now appear below.
2. Just edit the content
3. After you have edited the content, you can type `:d <register>` to put back the content

e.g. The followed gif shows the procedure of modifying a macro. I have a dumb macro in the register `y` whose function is appending ` world` in the current line. I just modify it to append ` hello world` instead.

![](/img/modify_macro.gif)

## Combined with other tools
In the Linux world, we can combine the CLI tools to achieve what we want in an elegant way.

Let's say you want to edit a bunch of files(the operation is the same) using `vim`. Check this [link](https://stackoverflow.com/questions/4867679/redirecting-output-of-find-command-to-vim).

1. You can using `find <pattern> -exec vim {} +` to open all the files in `vim`.
2. Use the `bufdo` technology aforementioned to modify files.

## Wrap up
This is a simple tutorial about the macro in vim. It is just the tip of the iceberg in `vim`. Only when you really master the `vim` you may make the most of it. It would be great if this blog get you interested in `vim`. I once heard that: at first, we choose the tools we like, and then the tool will shape us. Hope this tutorial will be helpful 😉

