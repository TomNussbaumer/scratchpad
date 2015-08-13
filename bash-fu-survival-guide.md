# Bash Fu: The Commandline Survival Guide

## Preface

```shell
for DVDs in Linux screw the MPAA and ; do dig $DVDs.z.zoy.org ; done | \
  perl -ne 's/\.//g; print pack("H224",$1) if(/^x([^z]*)/)' | gunzip 
```

... now figure out what's going on. It's [one of the coolest hacks](http://decss.zoy.org/) I've ever seen ...

## Table of Content

  * [Introduction](#introduction)
  * [History Goodness](#history-goodness)
    * [History related builtin commands](#history-related-builtin-commands)
    * [Examples](#examples)
  * [Standard features](#standard-features)
  * [Special options / flags of Bash](#special-options--flags-of-bash)
  * [Special variables (incomplete list)](#special-variables-incomplete-list)
    * [Positional Parameters](#positional-parameters)
  * [Brace expansion](#brace-expansion)
  * [Useful tools and builtins](#useful-tools-and-builtins)
  * [IFS (inter field separator) tricks](#ifs-inter-field-separator-tricks)
  * [The Magic of Cron/Crontab](#the-magic-of-croncrontab)
  * [Oneliners (Generic)](#oneliners-generic)
  * [Oneliners (Networking)](#oneliners-networking)
  * [File and filesystem oneliners](#file-and-filesystem-oneliners)

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

## Standard features

```shell
# looping
# 
# for VAR in LIST; do .......; done
# while EXPR;      do .......; done

# greetings to everyone
for FRIEND in Anna Fred Sophia; do
   echo "Hello, $FRIEND!"
done

# read file line by line
while read LINE; do
   echo $LINE
done < inputfile.txt

# ping a host until it is up
while true; do
  ping -c 1 -W 1 remote-host >/dev/null 2>&1 && break
done
echo "Remote-host is up at $(date)."

# command substitution
# 
# VAR=$(command)    or    VAR=`command`
for VAR in $(ls -1); do echo $VAR; done 
ps -fp $(cat /var/run/crond.pid)
```

## Special options / flags of Bash

There are many more flags and options, but the most commonly used ones are below. Type `help set' in a bash shell to see more.

```shell
# print commands and their arguments as they are executed
set -x

# exit immediately if a command exits with a non-zero status
set -e

#  the return value of a pipeline is the status of
#  the last command to exit with a non-zero status,
#  or zero if no command exited with a non-zero status
set -o pipefail
```

## Special variables (incomplete list)

**$BASH**&nbsp;&nbsp;&nbsp;&nbsp;path to Bash binary
```shell
echo $BASH  # prints: /bin/bash
```

**$BASH_SUBSHELL**&nbsp;&nbsp;&nbsp;&nbsp;indicates the subshell level
```shell
# prints lvl=1 lvl=2 lvl=3
echo lvl=$BASH_SUBSHELL $(echo lvl=$BASH_SUBSHELL $(echo lvl=$BASH_SUBSHELL))"
```
**$PPID**&nbsp;&nbsp;&nbsp;&nbsp;process id of parent<br/>
**$$**&nbsp;&nbsp;&nbsp;&nbsp;process id of toplevel shell<br/> 
**$BASHPID**&nbsp;&nbsp;&nbsp;&nbsp;process id of actual subshell<br/>
```shell
echo "top=$PPID:$$:$BASHPID" && (echo "sub=$PPID:$$:$BASHPID")
```

**$BASH_VERSION**&nbsp;&nbsp;&nbsp;&nbsp;version string of bash<br/>
**$BASH_VERSINFO[n]**&nbsp;&nbsp;&nbsp;&nbsp;array containing version infos as separated fields<br/>
```shell
echo "\$BASH_VERSION = $BASH_VERSION"
for i in {0..5}; do echo "\$BASH_VERSINFO[$i] = ${BASH_VERSINFO[$i]}"; done 
```

**$LINENO**&nbsp;&nbsp;&nbsp;&nbsp;actual line number in script<br/>
**$PIPESTATUS**&nbsp;&nbsp;&nbsp;&nbsp;array holding  exit statuses of each pipe path<br/>
```shell
# note the error in the last sed
echo "hello world!" | sed 's/hello/goodbye/' | sed 's/'
echo ${PIPESTATUS[*]}  # prints 0 0 1
# $PIPESTATUS is only available directly after the pipe(s). 
echo ${PIPESTATUS[*]} # prints 0
```

**$PWD**&nbsp;&nbsp;&nbsp;&nbsp;current working directory (same as output of command _pwd_ )
```shell
echo "PWD=$PWD"
echo "pwd=$(pwd)"
```

**$UID**&nbsp;&nbsp;&nbsp;&nbsp;user id of user executing the script
```shell
#!/bin/bash
# am-i-root.sh: determines if user is root
if [ "$UID" -eq "0" ];then
  echo "You are root."
  exit 0
fi
echo "Just an ordinary user."
exit 1
```

**$?**&nbsp;&nbsp;&nbsp;&nbsp;exit code of latest executed command, function or script. The exit code of a program should be 0 if it executed correctly.
```shell
[ "abc" = "abc" ]; echo "result of [abc=abc] is $?"
[ "abc" = "def" ]; echo "result of [abc=def] is $?"
```

**$!**&nbsp;&nbsp;&nbsp;&nbsp;process id of lastest started background job
```shell
CMD="sleep 10"
$CMD &
echo "pid of sleep=$!"
```

**$_**&nbsp;&nbsp;&nbsp;&nbsp;final argument of previously executed command. Note: the variable may not hold what you expect if the executed command is an alias.
```shell
ls >/dev/null
echo "final arg of [ls]     = $_"
ls -al >/dev/null
echo "final arg of [ls -al] = $_"
```

### Positional Parameters

1. **$0, $1, $2, etc.**&nbsp;&nbsp;&nbsp;&nbsp;Positional parameters passed from command line to script or passed to a function
2. **$#**&nbsp;&nbsp;&nbsp;&nbsp;Number of command line arguments or positional parameters
3. __$*__&nbsp;&nbsp;&nbsp;&nbsp;All positional parameters seen as a single word (must be quoted!)
4. **$@**&nbsp;&nbsp;&nbsp;&nbsp;Same as $*, but each paramter is a quoted string (must be quoted!)

Command _shift_ can be used to remove the first or more parameters from _$@_.

## Brace expansion

```shell
# make a backup - executes: cp test.log test.log.bak
cp test.log{,.bak}

# prints: abc.bin abc.lib abc.log
echo abc{.bin,.lib,.log}

# sequence - prints: 10.0.0.1 10.0.0.2 10.0.0.3
echo 10.0.0.{1..3}

# sequence with increment - prints: abc0def abc5def abc10def
echo abc{0..10..5}def

# sequence with padding - prints: abc000def abc001def abc002def
echo abc{000..002}def

# backward sequence - prints: x9 x8 x7
echo x{9..7}

# character sequence - prints: a b c d e
echo {a..e}

# character sequence backwards - prints: z x
echo {z..x}

# character sequence with increment - prints: a c e
echo {a..e..2}

# multiple expansions - prints: a1.txt a2.txt a3.txt b1.txt b2.txt b3.txt
echo {a..b}{1..3}.txt
```

**NOTE:** Brace expansion is the very first step of the shell's expansions. **Variables will be replaced by their values after brace expansion is done.** So you can't use dynamic sequences like this:

```shell
# pitfall: loop runs only once and prints: {1..3}
start=1
end=3
for i in {$start..$end}; do echo $i; done
```

## Useful tools and builtins

tool             | description
---------------- | ---------------------------------------------
at               | schedule jobs for later execution
awk              | line-oriented processing engine
crontab          | schedule regular jobs
curl             | HTTP client (crawl URL)
find             | recursive filesystem traversal
iconv            | converts files from one encoding to another
lsof             | list of files (everything is a file in linux)
nohup            | keeps command running after disconnect
screen           | screen multiplexer (detach, re-attach to sessions)
script           | saves copy of terminal session
sed              | stream editor (parse and transform)
sort             | sort input by fields
time             | displays time statistics of given prg after run
tr               | translate characters in stream
uniq             | omit adjacent duplicate lines
wget             | HTTP client

## IFS (inter field separator) tricks

```shell
# extract fields from passwd file
# (NOTE: manipulation of IFS in subshell doesn't affect current shell)
cat /etc/passwd | ( \
			IFS=: ; while read lognam pw id gp fname home sh; \
				do echo $home \"$fname\"; done \
				)

# sort input while leaving header line intact
# (append function body to your .bashrc and you can use it everywhere)
body() {
  IFS= read -r HEADER
  printf '%s\n' "$HEADER"
  "$@"
}
df -h | body sort -k 6
```

## The Magic of Cron/Crontab

Cron is a job scheduler with can be found on almost all unix-like systems. It's basic usage is to execute jobs at specified times.

The configuration of cron is stored in files as a table (hence crontab). The format of a single configuration row is:

```
MIN HOUR DAY MON DOW CMD
```

The following table describes the fields and their allowed values:

Field | Description   | Allowed Value
----- | ------------- | -------------
MIN   | Minute field  | 0 to 59
HOUR  | Hour field    | 0 to 23
DAY   | Day of Month  | 1 to 31
MON   | Month field   | 1 to 12
DOW   | Day Of Week   | 0 to 6 (0==Sunday)
CMD   | Command       | command to execute

The configuration of a job which should execute on August 1st at 17:30 will look like this:

```
30 17 1 8 * /home/johndoe/important-background-job.sh
```

The numeric fields can hold single values, lists and/or ranges:

```
# will execute at minute 11, 23 and 57 every hour
11,23,57 * * * * /home/johndoe/irregular-minutes.sh

# will execute every hour from 09:00 to 18:00 during weekdays (excl. the weekend)
00 09-18 * * 1-5 /home/johndoe/working-hours.sh

# will execute every 5 minutes
*/5 * * * * /home/johndoe/every-5-minutes.sh

# will execute every 2 minutes in the first 10 minutes of each hour
0-10/2 * * * * /home/johndoe/every-2nd-minutes-of-0-to-10.sh
```

Additionally to the above specifications you can also use the following special keywords: 

Keyword   | Equivalent   | Meaning
--------- | ------------ | ---------------------------------
@yearly   | 0 0 1 1 *    | every January 1st at 0:00
@monthly  | 0 0 1 * *    | every first day of month at 0:00
@weekly   | 0 0 * * 0    | every sunday at 0:00
@daily    | 0 0 * * *    | every day at 0:00
@hourly   | 0 * * * *    | every hour at minute 0
@reboot   | none         | run at startup


Command **crontab** can be used to view and manipulate cron jobs. Editing crontab entries boils down to use a text editor to make any changes to the crontab file.

**IMPORTANT NOTE:** if you use the -r option of crontab for removal, it will remove the complete file which means all your scheduled jobs are gone!!

Cron has one major drawback: if doesn't check if it has missed to run a scheduled job (i.e. due to a shutdown). If you need garanteed executions you have to install and use something like [Anacron](https://www.google.at/search?q=anacron). 


## Oneliners (Generic)

Reset your terminal emulator (if there are weird chars on your screen):

```shell
reset
```

Top 10 list of your most used commands (unix piping demo):

```shell
history | awk '{print $2}' | sort | uniq -c | sort -rn | head
```

Strip out comments and blank lines:

```shell
# -v invert matches
# -E use regular expression
grep -v -E "^#|^$" a_text_file
```

Format output as table:

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

Extract the Nth line from a file:

```shell
# extracts line 10 from inputfile
awk 'NR==10' inputfile
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

set up multiple variables at once:

```shell
#NOTE: piping is not possible because cmds in a pipe run in subshells
read a b c <<<$(echo "10 20 30")
```

Sum all numbers in a column:

```shell
# cat file |  awk '{ sum += $1 } END { print sum }

AVAILABLE=$(df -BG | grep -v Avail | awk '{ sum += $4 } END { print sum }')
echo "available on all mounted filesystems: $AVAILABLE GB"

```

List processes:

```shell
# top 5 cpu consumers
ps aux | head -1 && ps aux | sort -nk 3 | tail -5

# top 5 memory consumers
ps aux | head -1 && ps aux | sort -nk 4 | tail -5
```

Get Linux kernel information:

```shell
uname -a
```

Generate random password:

```shell
#openssl rand -base64 48 | cut -c1-PASSWORD_LENGTH
openssl rand -base64 48 | cut -c1-6
```

Rudimentary commandline stopwatch using time:

```shell
# press enter a second time when you want it to stop and show result
time read
```

Repeat a command periodically and watch it's output change:

```shell
watch date
watch df
```

Execute a command at a given time:

```shell
TESTFILE=$(pwd)/test.txt
echo "touch $TESTFILE" | at NEXT MINUTE
watch ls -l test.txt

# shows job details after adding it
JOBNR=$((echo "touch $TESTFILE" | at NEXT MINUTE 2>&1) | grep 'job' | awk '{print $2}')
at -c $JOBNR
```

## Oneliners (Networking)

Use local vim to edit files on remote host:

```shell
vim scp://remote-host//path/to/file
vim scp://remote-user@remote-host//path/to/file
```

Mount a directory from remote server via SSH:

```shell
# mount
sshfs remote-user@remote-host:/directory mountpoint
# unmount
fusermount -u mountpiont
```

Get own public IP address/name:

```shell
# for more see:
#   https://www.ipify.org
#   http://www.statdns.com/api
IP=$(curl https://api.ipify.org)
curl http://api.statdns.com/x/$IP

```

Show open network connections:

```shell
sudo lsof -Pni
```

Compare difference between remote and local file:

```shell
ssh remote-host cat /path/to/remotefile | diff /path/to/localfile -
```

Send a file as email attachment:

```shell
echo "see attachment" | mail -s "the requested file" -a /tmp/files.tgz tom@mycorp.net
```

SSH tunneling (local port forwarding):

```shell
# forward local port 9000 to google.com port 80 using remote host as man-in-the-middle
ssh -nNT -L 9000:google.com:80 user@remotehost

# forward local port 9000 to localhost port 2222 of remote host
ssh -nNT -L 9000:localhost:2222 user@remotehost
```

SSH tunneling (remote port forwarding):

```shell
# forwards port 9000 of remotehost to localhost 3000
# (requires "GatewayPorts yes" in /etc/ssh/sshd_config)

ssh -R 9000:localhost:3000 user@remotehost
```

Dynamically proxying via SSH for secure surfing on open wireless hotspots:

```shell
# proxies all connections to 1080 via ssh to remote host
# configure your browser to use that port and you are done
ssh -ND 1080 user@remotehost
```

Which programs listen on which ports:

```shell
sudo netstat -nutlp
```

Disconnect from a remote session and reconnect later:

```shell
ssh remotehost
screen
# start some work
top
# === press <ctrl-a> <d> ===
exit
# reconnect
ssh remotehost
screen -r
```

Run a command immune to hangups:

```shell
ssh myserver
nohup important-script.sh &
exit
ssh myserver
cat nohup.out
```
## File and filesystem oneliners

Follow a file as it grows:

```shell
tail -f /var/log/syslog
```

Follow multiple files as they grow:

```shell
# you may need to install it first
sudo multitail /var/log/syslog /var/log/kern.log
```

Print a list of files that contain a string:

```shell
sudo grep -lr ERROR /var/log
```

Recursive filesystem traversal (good if there are tons of files):

```shell
# NOTE: you can do all kind of stuff with find like filtering
#       and/or applying commands to found files
find . -type f -ls

# find all logfiles in and below your home dir containing "ERROR"
find ~ -type f -name '*.log' -exec grep -q "ERROR" '{}' \; -print

```

Convert a file from one encoding to another (nowadays unix2dos isn't enough):

```shell
# Windows-1252 encoding -> UTF8
iconv -f WINDOWS-1252 -t UTF-8 inputfile
```

Change to previous directory:

```shell
cd -
```

Using find (more examples):

```shell
# find by fixed name
find . -name "MyTest.log"

# ignoring character cases
find . -iname "MyTest.log"

# with limited search depth
find / -maxdepth 2 -name passwd

# with min/max search depth
find / -mindepth 2 -maxdepth 2 -name passwd

# with a special type (tpye=directory in the following case)
find . -type d

# executing commands
find . -maxdepth 1 -type f -exec md5sum {} \;

# invert next match option (multiple match options are applied after each other)
find /var/log -maxdepth 1 -not -type d -not -name "*.gz" -not -name "*log*"

# find by file permissions and list them
# note: the following find returns files with are writable by everyone in /etc
#       !! it should not find anything !!
find /etc -maxdepth 1 -perm -a=w -exec ls -l {} \;

# find top 5 largest files in your home directory
find ~ -type f -exec ls -s {} \; | sort -n -r | head -5

# bigger than a given size
find ~ -size +100M

# small than a given size
find ~ -size +100M

# find files which match time characteristics

find . -mmin 60    # modified in the last hour
find . -mtime 1    # modified in the last 24 hours (= 1 day)
find . -amin 60    # accessed in the last hour (may be disabled for a filesystem!)
find . -atime 1    # accessed in the last 24 hours
find . -cmin 60    # changed in the last hour
find . -ctime 1    # changed in the last 24 hours

# find files modified after modification of given file
find . -newer /etc/hosts

# find files accessed after modification of given file
find . -anewer /etc/hosts

# find files which status has changed after modification of given file
find . -cnewer /etc/hosts

#---------------------------------------------------------------------
# That's a very useful one:
#
# find only on given filesystem (don't descent into mounts)
#---------------------------------------------------------------------
find / -xdev -size +100M

#---------------------------------------------------------------------
# Another very useful one:
#
# executing TWO finds in ONE run and print result to different files
#---------------------------------------------------------------------
find /var/log  \( -name '*.log' -fprintf ./ext-log.txt '%-10s %p\n' \) , \
               \( -name '*.gz'  -fprintf ./ext-gz.txt  '%-10s %p\n' \)
```
