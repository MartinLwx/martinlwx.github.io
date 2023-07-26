# The solutions for exercise 05&06 of MIT.Missing-semester(2020)


## Lecture05. Command-line Environment

---

### Job control

> From what we have seen, we can use some `ps aux | grep` commands to get our jobs’ pids and then kill them, but there are better ways to do it. Start a `sleep 10000` job in a terminal, background it with `Ctrl-Z` and continue its execution with `bg`. Now use [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) to find its pid and [`pkill`](http://man7.org/linux/man-pages/man1/pgrep.1.html) to kill it without ever typing the pid itself. (Hint: use the `-af` flags).

```bash
> sleep 10000
# press Ctril-Z
# output: [1]  + 29705 suspended  sleep 10000

> bg %1
# output: [1]  + 29705 continued  sleep 10000

> pgrep -af "sleep"
# output: 29705

> pkill -af "sleep"
# output: [1]  + 29705 terminated  sleep 10000
```

Apparently, `pgrep` and `pkill` are quite handy. We won't be bothered to find out the `pid` ourselves.

> Say you don’t want to start a process until another completes. How would you go about it? In this exercise, our limiting process will always be `sleep 60 &`. One way to achieve this is to use the [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) command. Try launching the sleep command and having an `ls` wait until the background process finishes.
>
> 
>
> However, this strategy will fail if we start in a different bash session, since `wait` only works for child processes. One feature we did not discuss in the notes is that the `kill` command’s exit status will be zero on success and nonzero otherwise. `kill -0` does not send a signal but will give a nonzero exit status if the process does not exist. Write a bash function called `pidwait` that takes a pid and waits until the given process completes. You should use `sleep` to avoid wasting CPU unnecessarily.

We can use `wait <pid>` to wait for a process to complete before proceeding. Then the question is what is exactly the `pid` of `sleep 60 &`. In last exercise, we know we can use `pgrep` to get the pid by process name. So the basic solution will be:

```bash
> sleep 60 &
> pgrep "sleep" | wait && ls
```

However, the `wait` command is session-dependent. So we need to use `kill -0`. The `pidwait()` in `pidwait.sh` is like :arrow_down:

```bash
#!/bin/bash
pidwait() {
    while kill -0 $1 2>/dev/null  # catch stderr to /dev/null
    do
        sleep 1
    done
    ls
}
```

You can test like this :arrow_down:

```bash
> source pidwait.sh
> sleep 60 &
> pidwait $(pgrep "sleep")
```

### Terminal multiplexer

> Follow this `tmux` [tutorial](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) and then learn how to do some basic customizations following [these steps](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)

Just follow the tutorial.

### Aliases

> Create an alias `dc` that resolves to `cd` for when you type it wrongly.

```bash
> alias dc=cd
```

> 1. Run `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` to get your top 10 most used commands and consider writing shorter aliases for them. Note: this works for Bash; if you’re using ZSH, use `history 1` instead of just `history`.

```bash
> alias top10="history 1| awk '{\$1=\"\";print substr(\$0,2)}' | sort | uniq -c | sort -n | tail -n 10"
```

:warning: WARNING: You need to use `\` to escape `"` and `$n`

### Dotfiles

> Let’s get you up to speed with dotfiles.
>
> 1. Create a folder for your dotfiles and set up version control.
> 2. Add a configuration for at least one program, e.g. your shell, with some customization (to start off, it can be something as simple as customizing your shell prompt by setting `$PS1`).
> 3. Set up a method to install your dotfiles quickly (and without manual effort) on a new machine. This can be as simple as a shell script that calls `ln -s`for each file, or you could use a [specialized utility](https://dotfiles.github.io/utilities/).
> 4. Test your installation script on a fresh virtual machine.
> 5. Migrate all of your current tool configurations to your dotfiles repository.
> 6. Publish your dotfiles on GitHub.

It is weird. :thinking: I created a folder and migrate all my configurations to it. And I also made soft links. However, it is just didn't work.

### Remote machines

> Install a Linux virtual machine (or use an already existing one) for this exercise. If you are not familiar with virtual machines check out [this](https://hibbard.eu/install-ubuntu-virtual-box/) tutorial for installing one.
>
> 1. Go to `~/.ssh/` and check if you have a pair of SSH keys there. If not, generate them with `ssh-keygen -o -a 100 -t ed25519`. It is recommended that you use a password and use `ssh-agent` , more info [here](https://www.ssh.com/ssh/agent).
>
> 2. Edit `.ssh/config` to have an entry as follows
>
>    ```
>     Host vm
>         User username_goes_here
>         HostName ip_goes_here
>         IdentityFile ~/.ssh/id_ed25519
>         LocalForward 9999 localhost:8888
>    ```
>
> 3. Use `ssh-copy-id vm` to copy your ssh key to the server.
>
> 4. Start a webserver in your VM by executing `python -m http.server 8888`. Access the VM webserver by navigating to `http://localhost:9999` in your machine.
>
> 5. Edit your SSH server config by doing `sudo vim /etc/ssh/sshd_config`and disable password authentication by editing the value of `PasswordAuthentication`. Disable root login by editing the value of `PermitRootLogin`. Restart the `ssh` service with `sudo service sshd restart`. Try sshing in again.
>
> 6. (Challenge) Install [`mosh`](https://mosh.org/) in the VM and establish a connection. Then disconnect the network adapter of the server/VM. Can mosh properly recover from it?
>
> 7. (Challenge) Look into what the `-N` and `-f` flags do in `ssh` and figure out a command to achieve background port forwarding.

I don't have enough disk space for a linux virtual machine :(

## Lecture 06. Version Control(git)

---

> If you don’t have any past experience with Git, either try reading the first couple chapters of [Pro Git](https://git-scm.com/book/en/v2) or go through a tutorial like [Learn Git Branching](https://learngitbranching.js.org/). As you’re working through it, relate Git commands to the data model.

:)

> Clone the [repository for the class website](https://github.com/missing-semester/missing-semester)
>
> 1. Explore the version history by visualizing it as a graph.
> 2. Who was the last person to modify `README.md`? (Hint: use `git log` with an argument).
> 3. What was the commit message associated with the last modification to the `collections:` line of `_config.yml`? (Hint: use `git blame` and `git show`).

```bash
> git clone git@github.com:missing-semester/missing-semester.git
> cd missing-semester

# step 1.
> git log --oneline --all --graph --color          

# step 2.
> git log -1 README.md			# -1 means showing last related commit
# then we know Anish Athalye is the last person modify README.md

# step 3.
> git blame _config.yml | grep -E "collections:"
# then we know the commit's sha-1 value is a88b4eac
# we can use `git show` command to get commit message
> git show -q a88b4eac			# -q means --quiet, it will suppress diff output
```

> One common mistake when learning Git is to commit large files that should not be managed by Git or adding sensitive information. Try adding a file to a repository, making some commits and then deleting that file from history (you may want to look at [this](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)).

I made a new repo and made some commits. Finally I found the correct command to delete a file from all history :hugs:

Ths command we are going to use is `git filter-repo`. If you haven't installed it, you may just run `brew install git filter-repo `(for mac users). Then use this "magic" command do finish this exercise.

```bash
> git filter-repo --invert-paths --force --path <file>
```

The reason why I use `--force` is that `git filter-repo` refuses to overwrite my repo. It says my repo doesn't look like a fresh clone.

The mechanism behind this command is: `--invert-paths` will only select files matching none of options which we specify by `--path`. So it will keep everything except `<file>`, and overwrite this commit history.

> Clone some repository from GitHub, and modify one of its existing files. What happens when you do `git stash`? What do you see when running `git log --all --oneline`? Run `git stash pop` to undo what you did with `git stash`. In what scenario might this be useful?

I tried to modify `README.md` is the repo for the class website. 

Then I ran `git stash`. The output is: `Saved working directory and index state WIP on master: c2f9535 Merge pull request #172 from chapmanjacobd/patch-1`

```
*   96cca4b (refs/stash) WIP on master: c2f9535 Merge pull request #172 from chapmanjacobd/patch-1
|\
| * a95c46f index on master: c2f9535 Merge pull request #172 from chapmanjacobd/patch-1
|/
*   c2f9535 (HEAD -> master, origin/master, origin/HEAD) Merge pull request #172 from chapmanjacobd/patch-1
```

Then I ran `git stash pop`. My modification has come back.

The scenario of using `git stash` is that: For exmaple, say I have beening working on my project in `master` branch for quite a long time. Now I want to switch to another branch, but my working directory is in a mess. I don't want to make a dirty commit to temporarily save my work. Here comes the `git stash`. **I can use this command to temporarily save my half-finished work.** :hugs:

> Like many command line tools, Git provides a configuration file (or dotfile) called `~/.gitconfig`. Create an alias in `~/.gitconfig` so that when you run `git graph`, you get the output of `git log --all --graph --decorate --oneline`. Information about git aliases can be found [here](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias).

```bash
> git config --global --add alias.graph 'log --all --graph --decorate --oneline'
> git graph
```

> You can define global ignore patterns in `~/.gitignore_global` after running `git config --global core.excludesfile ~/.gitignore_global`. Do this, and set up your global gitignore file to ignore OS-specific or editor-specific temporary files, like `.DS_Store`.

```bash
> git config --global core.excludesfile ~/.gitignore_global
> echo ".DS_Store" >> ~/.test
```

> Fork the [repository for the class website](https://github.com/missing-semester/missing-semester), find a typo or some other improvement you can make, and submit a pull request on GitHub (you may want to look at [this](https://github.com/firstcontributions/first-contributions)).

I followed first-contributions in [this](https://github.com/firstcontributions/first-contributions).:happy:


