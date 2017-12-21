InWee
=====
InWee is a convenience wrapper script to send text messages and commands
to WeeChat from shell or from within WeeChat via
[WeeChat's FIFO pipe][1].

[![Download][SHIELD]][DOWNLOAD]

[1]: http://www.weechat.org/files/doc/stable/weechat_user.en.html#fifo_plugin
[SHIELD]: https://img.shields.io/badge/download-inwee-brightgreen.svg
[DOWNLOAD]: https://github.com/susam/inwee/releases/download/0.2.0/inwee


Contents
--------
* [Necessity](#necessity)
* [Installation](#installation)
  * [Manual](#manual)
  * [With wget](#with-wget)
  * [With git](#with-git)
* [Getting Started](#getting-started)
  * [Using from Shell](#using-from-shell)
  * [Using from WeeChat](#using-from-weechat)
* [Without InWee](#without-inwee)
* [Credits](#credits)
* [License](#license)
* [Contact](#contact)


Necessity
---------
WeeChat keeps configuration in several .conf files in ~/.weechat
directory. The configuration can be edited with WeeChat `/set` commands.
Unfortunately, the same .conf files contain both the default
configuration options that have not been altered by the user as well as
custom configuration options specified by the user. This makes it
clumsy to pick only the user specific custom configuration, save it in a
separate file as backup, so that it can be applied later when needed,
say after reinstallation of operating system or on a new system.

One solution to this problem is to keep all the user specific
configuration in the form of `/set` commands in a file, say
settings.txt, as shown below.

     /set irc.server.freenode.nicks humpty,humpty_
     /set irc.server.freenode.username humpty
     /set irc.server.freenode.realname "Humpty Dumpty"
     /set irc.server.freenode.autojoin #weechat,#test,#freenode
     /save irc
     /print Saved configuration

This file can be fed into WeeChat's FIFO pipe while WeeChat is running
using the following shell command.

    sed 's/^/*/' settings.txt > ~/.weechat/weechat_fifo_*

The command first prefixes every line in the file with `*` (asterisk)
because that is how WeeChat's FIFO pipe expects the input to be. Then it
feeds the lines to WeeChat's FIFO pipe.

If the above solution is good enough for you, then there is no real
reason to use InWee. In that case, you can skip directly to the
[Without InWee](#without-inwee) section and read about how everything
that can be done with InWee can also be done without it.

InWee only makes the above solution more convenient. For example, the
commands in settings.txt can be fed to WeeChat with InWee by entering
the following command while WeeChat is running.

    inwee settings.txt

This simple command offers the following advantages over the earlier
command involving sed command's output redirection.

 1. It is simpler and easier to type.
 2. It automatically prefixes every command in the input file with `*`
    (asterisk).
 3. It automatically checks if the FIFO pipe exists at ~/.weechat. If
    it does not exist, it quits with an error.
 4. It supports comments in the input file. Any line in the file where
    `#` (hash) is the first non-whitespace character is ignored as
    comment.

WeeChat can execute external commands with the `/exec` command and it
accepts the `-r` option to run commands after startup, so when WeeChat
is not running, it can be made to run the commands in settings.txt after
starting up as follows.

    weechat -r '/exec inwee settings.txt'


Getting Started
---------------
InWee is a single-file executable script: [`inwee`][DOWNLOAD].

Copy it to a directory specified in the PATH environment variable and
make it executable: `chmod u+x inwee`. In many Linux systems,
`/usr/local/bin` or `~/bin` is a good location to copy this script to.

InWee may be invoked directly from the shell using the `inwee` command
or from WeeChat using its `/exec` command. Several examples of both
ways of usage are provided below.

### Using from Shell ###
The following list shows various ways to use InWee from the shell.

 1. Here is a typical example of using InWee from the shell. Say
    you are in a Linux support channel such as ##linux, #debian,
    #centos, etc. and you need to share the network controller details
    of your system with the channel while asking for help. To do this
    with InWee, you can run the commands you need to get this
    output and pipe it into `inwee` as shown below to send the output
    to the current buffer in WeeChat.

        lspci | grep -i network | inwee

 2. Here are some general examples of sending text or commands from
    standard input to the current buffer in WeeChat. Any text or
    WeeChat command you pipe into the standard input of InWee is sent
    to WeeChat.

        echo hello world | inwee
        echo '/join #test' | inwee
        echo /part | inwee
        echo /nick newnick | inwee

    Note that comments in shell begin with `#`, therefore the WeeChat
    `join` command in the second shell command above is quoted. Any
    text beginning with `/` (slash), followed by a word and whitespace
    is considered as a WeeChat command. All other text is considered as
    text messages to be sent on chat.

 3. Multiple texts and/or commands must be separated by a newline.

        printf '/join #test\n/join #flood\n' | inwee

    Every line of text or command must also end with a newline. Since
    `printf` does not append a newline in the end like `echo` does, we
    end the strings passed to `printf` in each command above with a
    newline.

    Blank lines, lines that consist only of whitespace characters and
    lines that contain `#`, i.e. hash, as the first non-whitespace
    character are ignored.

 4. The previous example may also be conveniently written in the
    following manner.

        echo '/join #flood
        /join #test' | inwee

 5. Text or commands may be sent to a specific WeeChat buffer with the
    `-b` or `--buffer` option.

        echo hello world | inwee -b 'irc.freenode.#test'

    With the above command, the text is sent to #test channel even when
    this channel is not in the current buffer.

 6. Send text or commands from a file by specifying the path to the
    file as an argument.

        inwee input.txt

    The file may contain one or more lines of text or commands. Blank
    lines, lines that consist only of whitespace characters and lines
    that contain `#`, i.e. hash, as the first non-whitespace character
    are ignored. Therefore, `#` can be used to begin single line
    comments.

 7. To use a custom FIFO path with WeeChat.

        echo "hello world" | inwee --fifo "$HOME/.weechat/weechat_fifo"

    This will make inwee use a custom FIFO path to communicate with
    WeeChat.

### Using from WeeChat ###
From WeeChat's perspective, InWee is an external command that can be
invoked with the `inwee` command. WeeChat's `/exec` command executes
external commands inside WeeChat and displays the output locally, or
sends it to a buffer. Therefore, `inwee` can be used within WeeChat
using WeeChat's `/exec` command. This section shows a few such examples.
Note that unlike in the previous section where the command examples are
meant to be run in the shell, the commands below are meant to be run
inside WeeChat directly.

 1. Send text or commands from a file to the current buffer.

        /exec inwee input.txt

 2. Send text or commands from a file to #test channel.

        /exec inwee -b irc.freenode.#test input.txt


Without InWee
-------------
Everything that can be done with InWee can be done without it, although
it can be a little clumsy for some cases. This section is written for
those who do not want to use InWee but do what can be done with InWee by
working directly with WeeChat's FIFO pipe or WeeChat's `/exec` command.
WeeChat's FIFO pipe is located at ~/.weechat by default and named as
`weechat_fifo_<PID>` where `<PID>` stands for the PID of the currently
running instance of WeeChat.

Assuming WeeChat is currently running and there is a FIFO pipe for
WeeChat at ~/.weechat, the list below explains how something that can be
done with InWee can also be done without it.

 1. Send the output of a shell command from shell to WeeChat.

    With InWee:

        lspci | grep -i network | inwee

    Without InWee:

        lspci | grep -i network | sed 's/^/*/' > ~/.weechat/weechat_fifo_*

 2. Send a message to a WeeChat channel from shell.

    With InWee:

        echo hello world | inwee

    Without InWee:

        echo '*hello world' > ~/.weechat/weechat_fifo_*

 3. Send a WeeChat command from shell to WeeChat.

    With InWee:

        echo /nick newnick | inwee

    Without InWee:

        echo */nick newnick > ~/.weechat/weechat_fifo_*

 4. Send text or command to a specific WeeChat buffer from shell.

    With InWee:

        echo hello world | inwee -b irc.freenode#test

    Without InWee:

        echo 'irc.freenode.#test *hello world' > ~/.weechat/weechat_fifo_*

 5. Send text or commands in a file to WeeChat from shell.

    With InWee:

        inwee input.txt

    Without InWee:

        sed 's/^/*/' input.txt > ~/.weechat/weechat_fifo_*

 6. Send the output of a simple shell command in WeeChat to a channel.

    With InWee:

        /exec -sh uname -a | inwee

    Without InWee:

        /exec -o uname -a

    The `-sh` option is necessary in the first command to ensure that
    the `/exec` command uses the shell to execute the specified shell
    command. This ensures that `|` is interpreted as the pipeline
    operator by the shell. Without the `-sh` option, the `/exec`
    command executes the specified command itself and it would
    interpret `|` as just another argument.

    The `-o` command is necessary in the second command to ensure that
    the output of the shell command is sent to the channel as chat
    message. Without this option, the output is only printed locally
    and not shared with the channel. There is no need of `-o` option in
    the first command because InWee takes care of sending the output as
    text message to the channel's buffer via WeeChat's FIFO pipe.

 7. Send output to a specific channel in WeeChat.

    With InWee:

        /exec -sh uname -a | inwee -b irc.freenode.#test

    Without InWee:

        /exec -sh -o -buffer irc.freenode.#test uname -a

 8. Send the output of a little more complex shell command, say
    involving pipeline operators, in WeeChat.

    With InWee:

        /exec -sh lspci | grep -i network | inwee

    Without InWee:

        /exec -sh -o lspci | grep -i network

 9. Send text or commands in a file to the current buffer in WeeChat.

    With InWee:

        /exec inwee input.txt

    Without InWee:

        /exec -sh sed 's/^/*/' input.txt > ~/.weechat/weechat_fifo_$(pidof weechat)


Credits
-------
Thank you, [Dom Rodriguez][C1], for adding support for WeeChat 1.7.

[C1]: https://github.com/shymega


License
-------
This is free and open source software. You can use, copy, modify,
merge, publish, distribute, sublicense, and/or sell copies of it,
under the terms of the MIT License. See [LICENSE.md][L] for details.

This software is provided "AS IS", WITHOUT WARRANTY OF ANY KIND,
express or implied. See [LICENSE.md][L] for details.

[L]: LICENSE.md


Contact
-------
To report bugs, suggest improvements, or ask questions, please create a
new issue at <http://github.com/susam/inwee/issues>.
