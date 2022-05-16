---
title: "Linux's commands and tricks I'm using in my daily job as a developer "
date: 2019-11-11T15:00:00Z
draft: false
description: "This is not a post from the series of those describing the `cd` command. It's just a list of commands and tricks I'm using (almost) every day."
---

Post was originally posted on [my old dev.to blog](https://dev.to/mateuszjarzyna/linux-s-commands-and-tricks-i-m-using-in-my-daily-job-as-a-developer-4cle)

This is not a post from the series of those describing the `cd` command. It's just a list of commands and tricks I'm using (almost) every day.

## Port forwarding

Sometimes I have to connect to database and of course I prefer to use my GUI manager (JetBrains DataGrip).
So, if security policy exist in your company and your database's port is not exposed you can execute

```bash
ssh -L{port on your PC}:localhost:{database's port} root@{server IP}
```

The command below will open port `3308` on your laptop and everything will be forwarded to `192.168.1.2:3306`

```bash
ssh -L3308:localhost:3306 root@192.168.1.2
```

`localhost` means that database is listening on `192.168.1.2`. You can type for example `192.168.3.77` and everything will be forwarded to `.3.77` server via `.1.2`.

## Edit file in VIM without `sudo`, but save with `sudo`

Have you ever edited some configs file and forgot to `sudo`? Me too... There is a trick to save the file anyway, just type in VIM:

```bash
:w !sudo tee %
```

[Explanation](https://stackoverflow.com/a/7078429/1775344)

## Go to beggining/end of line in terminal

If you wrote a very long command in the terminal it may take a long time before you return to the begging of the line to add missing `sudo`. And back to the end to add some parameters.
Press `crtl + a` to move to the begging and `crtl + e `to the end of the line in terminal.

## ll

Save few days in a year by typing `ll` instead of `ls -la`. Works on most Linux servers.

## Execute command you executed in the past

### Last command

To execute last command over again you can of course press ↑ (arrow up) key. But you can also type `!!`. So executing last command as a root is very easy

```bash
sudo !!
```

To run the last command that started with `apt` type `!apt`

### Search history

To find the command that contains `/tm`p you have executed in the past press `ctrl + r` and type `/tmp`. Press `ctrl + r` again for next result.
To show all commands or to search using regular expression use

```bash
history | grep "/tmp"
```

## Agree for everything

To say yes for each question you can use application called `yes`

```bash
yes | yum install curl
```

use `yes no` to say no and discard.

> WARNING
>
> As @patricnox notices [in the comment](https://dev.to/patricnox/comment/hidp) - using `yes` may do unexpected things. You can accidentally install 10 GB of dependencies or other things you don't want to do.

## Run a long-lasting process in the background and close the terminal

If you run a script that will end in 3 days, you don't have to wait with the terminal window open to end. You can run it using `nohup` command

```bash
nohup wget http://large-files.com/10gb-super-movie.avi &
```

`wget` works in the background, output is saved to `nohup.out` file in working directory.

## Checking who has stolen your favourite port

It's really annoying when you are trying to run nginx but you can't because there is already apache running and port 443 is busy.
So, how to determinate which process is listening on port 80:

```bash
$ netstat -tulpn | grep 80
tcp6       0      0 :::80                 :::*                   LISTEN     10177/java
```

`10177` is a pid you are looking for. Now execute

```bash
ps aux | grep 10177
```

for more details.

## Reading logs

Everyone knows that `less` is a very good way to read a logs files. But you can also read gziped logs without extracting!

```bash
less /var/log/my-app/my-app.log.2015.12.14.gz
```

## Live reading

```bash
tail -f /var/log/my-app/my-app.log | grep ERROR
```

The command above will show only new lines that contains `ERROR`.

## Sort process

Show top 3 processes sorted by CPU usage

```bash
ps aux --sort=-pcpu | head -n 4
```

Show top 3 processes sorted by memory usage

```bash
ps aux --sort=-rss | head -n 4
```

## Executing command every X seconds

To print command's output every X seconds you can use `watch` command. For example to create clock run

```bash
watch -n 1 date
```

## Quiet mode

A lot of standards commands has quiet or silent mode. Very useful when you are creating some bash script. In most of the cases just add `-q` or `-s` (read --help or man or check on StackOverflow)

```bash
zip -q archive.zip big-file.jpg
```

But sometimes (practically always with in-house scripts) you have to ignore the output (send to `/dev/null`)

```bash
./very-verbose.sh 1>/dev/null
```

## Create log files for scripts executed by crontab

```bash
0 22 * * 1-5 /opt/scripts/send-report.sh 2>/var/log/scripts/report-error.log
```

So next time when your script will fail you won't lose the reason
