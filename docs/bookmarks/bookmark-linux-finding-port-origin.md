---
title: 'Linux finding port origin'
date: '08:45 29-02-2024'
taxonomy:
    category:
        - docs
---

Bookmark
Source: [https://serverfault.com/a/847910](https://serverfault.com/a/847910)


Based on hint from @user202173 and others I have been able to use the following to track down the process that owns a port even when it is listed as - in netstat.

Here was my starting situation. sudo netstat shows port with PID/Program of -. lsof -i shows nothing.

```
$ sudo netstat -ltpna | awk 'NR2 || /:8785/'
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp6       0      0 :::8785                 :::*                    LISTEN      -
tcp6       1      0 ::1:8785                ::1:45518               CLOSE_WAIT  -
$ sudo lsof -i :8785
$
```

Now let's go fishing. First let's get the inode by adding -e to our netstat call.

```
$ sudo netstat -ltpnae | awk 'NR2 || /:8785/'
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode       PID/Program name
tcp6       0      0 :::8785                 :::*                    LISTEN      199179     212698803   -
tcp6       1      0 ::1:8785                ::1:45518               CLOSE_WAIT  0          0           -
```

Next use lsof to get the process attached to that inode.

```
$ sudo lsof | awk 'NR1 || /212698803/'
COMMAND      PID    TID                USER   FD      TYPE             DEVICE   SIZE/OFF       NODE NAME
envelope_ 145661 145766               drees   15u     IPv6          212698803        0t0        TCP :8785 (LISTEN)
```

Now we know the process id so we can look at the process. And unfortunately it's a defunct process. And its PPID is 1 so we can't kill its parent either (see How can I kill a process whose parent is init?). In theory init might eventually clean it up, but I got tired of waiting and rebooted.

```
$ ps -lf -p 145661
F S UID         PID   PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
0 Z drees    145661      1  2  80   0 -     0 exit   May01 ?        00:40:10 envelope <defunct>
```