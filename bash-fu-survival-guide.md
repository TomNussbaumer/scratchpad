# Bash Fu: The Commandline Survival Guide

## Introduction

Commandline shells came a long way and in many different flavours since the birth of Unix. This writeup focuses on ne of the most feature-rich and widely available shells known as [the Bash Shell](http://www.gnu.org/software/bash/manual/). 

Eventually (~1988) a standard arrived for Unix-like machines arrived: POSIX.

POSIX is a family of standards which defines:

  1. core programming interface
  2. commandline and scripting interface
  3. user-level programs, services, and utilities (ex.: awk,echo, ed)
  4. program-level services including basic I/O (ex.: file, terminal, network) 
  5. standard threading library API
  6. ... many more

When you want to write scripts which should run on a very large varity of systems, stick to the POSIX standards. Even if the Bash shell is available on your target systems, there are very good reasons to stick to the standards and don't use the Bash shell:

  1. reduced memory footprint
  2. less disk space required
  3. speed

Ubuntu, for example, uses Dash and POSIX-compliance for most of their system scripts. You can read about their reasoning [here](https://wiki.ubuntu.com/DashAsBinSh). 

So why not write all your scripts in the POSIX-compliant way?

It all boils down to one thing: comfort. The POSIX standards for scripts are a quite restricted feature set of what Bash gives you. There aren't even arrays in the standards. 

And, of course, writing POSIX-compliant scripts needs a mind switch. Using the bash interactively most of the day, you are used to get things done quickly which means with as less keystrokes as necessary. This almost always involves Bash extensions. When writing a POSIX-compliant script you need to switch your mind to a completely different mode, because all that nice shortcuts are gone.

## History Goodness

>History expansions introduce words from the history list into the input stream, making it easy to repeat commands, insert the arguments to a previous command into the current input line, or fix errors in previous commands quickly. 

### History related builtin commands

1. fc
2. history

### Examples

```shell
# list previous commands (truncated list)
fc -l

# list all previous commands
history

# list five latest commands
history | tail -5

# clear history
history -c

# repeat last command
echo 'hello world!'
!!

# repeat command from history index (you'll need another number!)
!1370

# repeat last command starting with given chars (searches backwards)
echo 'hello world!'
!ech

# reuse second word of prev. command
ls /etc
cd !^

# reuse last word of prev. command
ls -a /etc
cd !$
cd -   # additional TRICK: return to previous directory

# reuse the Nth word from prev. command ( !!:N )
ls -l /etc/hosts /etc/hostname
cat !!:2 

# reuse the Nth word from command starting with given chars ( !abc:N )
cat !ls:2

# repeat prev. command while removing a string ( ^str^ )
ls -a /xxxetc/hosts
^xxx^

# repeat prev. command while substituting a string ( ^str1^str2^ )
ls -a /rzc/hosts
^rz^et^

# repeat prev. command while substituting multiple times ( ^str1^str2^:& )
ls -a /rzc/hosts /rzc/hostname
^rz^et^:&

# reference a word of the current command and reuse it ( !#:N )
mv long_file_name some_prefix_!#:1.old
```

## Useful tools and builtins

tool             | description
---------------- | --------------------------------
awk              | line-oriented processing engine
script           | saves copy of terminal session
sed              | stream editor (parse and transform)
tr               | translate characters in stream

## Oneliners

Top 10 list of your most used commands:

```shell
history | awk '{print $2}' | sort | uniq -c | sort -rn | head
```

Strip out comments and blank lines:

```shell
# -v invert matches
# -E use regular expression
grep -v -E "^#|^$" a_text_file
```

Use local vim to edit files on remote host:

```shell
vim scp://remote-host//path/to/file
vim scp://remote-user@remote-host//path/to/file
```

Display out in a table:

```shell
echo -e 'one two\nthree four' | column -t
```

Extract words from output:

```shell
# extracts second word
echo 'one two three' | awk '{print $2}'

# extracts last word
echo 'one two three' | awk '{print $NF}'

# uses ':' as field separator
echo 'one:two:three' | awk -F: '{print $1,$NF}'

```

Append text to a file with sudo:

```shell
# generate file only writable by root
sudo touch demo.txt

# this doesn't work
sudo whoami >> test.txt

# this works!
echo "whoami" | sudo tee -a test.txt

# this works, too (subshell approach)
sudo bash -c 'whoami >> test.txt'
```

Display $PATH in a human readable form:

```shell
echo $PATH | tr ':' '\n'
```

Fast file creation without editor:

```shell
# press <ctrl-d> to close the file
cat > test.txt
```

Show special characters:

```shell
echo -e "\t\b\n" | cat -te
```

Extract multiple lines from a stream:

```shell
# first line of output will match start-pattern
# last line of output will match stop-pattern 
cat something | awk '/start-pattern/','/start-pattern/'
```

Delete a block between two Strings:

```shell
cat something | sed '/start-pattern/,/stop-pattern/d'
```

