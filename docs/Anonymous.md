---
title: /writeups/Anonymous
layout: post
permalink: /writeups/Anonymous
---
Writeup of THM Machine called <a href="https://tryhackme.com/r/room/anonymous">"Anonymous"</a>

# Enum

I started with the nmap scan:

![Nmap](/images/AnonymousNmap.png)

2 things caught my attention:
- Anonymous login to FTP
- Samba share

I decided to start with the FTP, but since the SMB is required to answer the 4th room question, I'll show it first.

To enumerate SMB further, I used the `smbmap` tool:

![SMB](/images/AnonymousSMB.png)

Of course, this share is a rabbit hole since there are only *cute* puppies.

Going back to the FTP share, we actually can Anonymously login and see several files in the /scripts directory.

![FTPfiles](/images/AnonymousFTP.png)

The most interesting one is the `clean.sh`

![clean.sh](/images/AnonymousClean.png)

It looks like the script deletes some files and then logs everything to the `removed_files.log` file. At this point I was already suspecting that this might be some cronjob, or another system timer, and to check if my assumptions are right, I simply looked into the .log file.

![Logs](/images/AnonymousLogs.png) 

Here we can clearly see that every minute or so, the file is appended with the message from the `clean.sh`

So I thought that the obvious way to exploit it, is to put some malicious code to the clean.sh or to replace it with our own file.
But it turned out that we weren't allowed to change permissions of the files uploaded by us, so I began to look for any way to edit the already existing file.
I found [this](https://askubuntu.com/questions/168300/edit-ftp-file-via-ubuntu-terminal) question on the askubuntu, and it worked perfectly.

Using the tool called `curlftpfs`. I ran this command, that mounts the FTP share to the local directory.
```bash
curlftpfs Anonymous@10.10.76.207 /home/silen/Desktop/THM/Anonymous/ftp 
```

And from there I simply edited the `clean.sh`, that it will connect to my listener, and give me an interactive bash shell. 

![Editing the script](/images/AnonymousScriptEdit.png)

In the meantime, I prepared the netcat listener.

![netcat](/images/AnonymousListener.png)

And got the foothold!
Now, there is only the PrivEsc left.


# PrivEsc

After quick enumeration, I found this interesting SUID binary.

![SUID](/images/AnonymousSUID.png)

the command I used:
```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

Looking up this binary in gtfobins, I found quick exploitation [here](https://gtfobins.github.io/gtfobins/env/).

![root](/images/AnonymousPWN.png)

Another machine PWNed!

Thanks for reading.
