# bash-terminal-sessions
Who needs tmux when you have bash?


https://github.com/vexorian/bash-terminal-sessions/assets/5443609/a6760ac0-7938-4b1a-b44f-765b3588d89f


## Features

Create bash "sessions" within your user. Each session has its own command history and if you close the session and open it again, it will remember the last work directory you used.

## Howto

Add the following to your `.bashrc` :

```bash
custom_cd() {
    builtin cd "$@"  # Call the real cd command
    echo "$PWD" > ~/.vx-bash.${VX_TERMINAL_ID}.pwd  # Save PWD to a file
}

if [ "$VX_TERMINAL_ID" != "" ]; then
    alias cd='custom_cd'

    cd $( cat ~/.vx-bash.${VX_TERMINAL_ID}.pwd )
    PS1="[${VX_TERMINAL_ID}] ${PS1}"
    HISTFILE=~/.vx.bash.${VX_TERMINAL_ID}.history
fi
```

Now, to start a session from inside a terminal:

```
VX_TERMINAL_ID=0 bash
```

The first time you use a bash session it will show an error about not finding the history file. Don't worry about it.

Your terminal prompt will now look like this;

```
[0] user@host:~$
```

This means that session `0` is used. The session id doesn't have to be a number. Though you probably don't want it to be very long or contain special characters. Definitely not `/`

Use the terminal as usual. Test some commands and use cd to go to specific places.

Then use EoF or type `exit` to close the session or close the terminal window.

Start a new terminal. Check that it doesn't remember the history from your session 0.

Restart the session by typing:

```
VX_TERMINAL_ID=0 bash
```

It will return the terminal to the work directory you were using and the history for session `0` will return.

Open a new terminal tab/window and type:

```
VX_TERMINAL_ID=1 bash
```


## How does it work?

I figure less interesting than the code is how it works. This knowledge should allow you to tweak the behavior as you wish.

### Environment var

bash config works by using the `.bashrc` file. For better or worse this is just another script file, one that happens to be ran when a bash instance starts. This means that you can use environment variables from inside the `.bashrc` file to modify the behavior of the configuration. In this case, we use an environment variable `VX_TERMINAL_ID` to signal to the bash rc that it should use a subsession and the id of that session.

Therefore, this if block:

```bash
if [ "$VX_TERMINAL_ID" != "" ]; then
    ...
fi
```

Is the one that will initialize the sub-session.

### cd hook

We want the session id to remember the working directory. This is not as simple as adding some sort of logic when bash is closed to save the current working directory. Because the bash instance could have been closed in many unexpected ways. Power failure being the one that you can't avoid at all. Instead, the method will add a hook so that everytime the user uses `cd`,  it will save the work folder in some file.

Bash allows you to create aliases for commands and these aliases can even override existing commands. So you can create an alias called `cd` that will be called instead of the `cd` command that we normally use to change the working folder.

```bash
alias cd='custom_cd'
```

This makes the `custom_cd` function to be called instead of `cd`.

```bash
custom_cd() {
    builtin cd "$@"  # Call the real cd command
    echo "$PWD" > ~/.vx-bash.${VX_TERMINAL_ID}.pwd  # Save PWD to a file
}
```

Although we are replacing `cd`, we still want the directory to be changed, so we use `builtin cd` to call the builtin cd command ignoring aliases.

The variable `$PWD` stores the current folder, so we save the contents of that variable in a file `~/.vx-bash.${VX_TERMINAL_ID}.pwd` . Note how the name of the file is diferent for each session id.


### History

bash history works by using a variable `$HISTFILE` to specify the location of a file where the history is saved. This variable can be changed to point to a different file. And that's what this line does:

```bash
HISTFILE=~/.vx.bash.${VX_TERMINAL_ID}.history
```

So now it will save in a different file depending on the session id.

### Command Prompt

The `PS1` variable determines the bash command prompt. Which is printed after every command so that it displays before you type another command. In this case we are modifying the `PS1` to append to it the name of the session:

```bash
PS1="[${VX_TERMINAL_ID}] ${PS1}"
```

## Tweaks

There are a number of things you can tweak.

The most obvious one should be to replace the name of the variable `VX_TERMINAL_ID` to something that's more to your liking.

There's something to be said about title bars. You might find it useful to include the sessioin id in the terminal's title bar or the tab name.

Changing the title from an interactive terminal works by writing a set of characters to stdout. This sequence is intercepted by the terminal and printed on the title window instead of the terminal itself. The trick bash uses by default is to add this sequence of text to the command prompt by modifying the `PS1` variable. This logic should be somewhere in the pre-defined contents of your .bashrc.

In my `.bashrc` it was like this:


```bash
# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac
```

So I tweaked this part to add the session id:

```bash
# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    if [ "$VX_TERMINAL_ID" != "" ]; then
        ADD=${VX_TERMINAL_ID}.
    fi
    PS1="\[\e]0;${ADD}${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac
```

So now it prints the session id plus a `.` as part of the window/tab title .


There are other things you could do. You could make a more aggressive replacement of the `PS1` variable so that you don't have to rely on existing `.bashrc` code. You can use an echo command yourself to print the title bar. Etc.



