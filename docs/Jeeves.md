---
title: /writeups/Jeeves
layout: post
permalink: /writeups/Jeeves
---
Writeup of HTB Machine called <a href="https://app.hackthebox.com/machines/Jeeves">"Jeeves"</a>

# Enum

I started with the nmap scan:

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
</pre></td><td class="rouge-code"><pre>$ nmap -sS -A -T4 -p- 10.10.10.63
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-08 12:03 CEST
Nmap scan report for 10.10.10.63
Host is up (0.31s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ask Jeeves
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2008 (86%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2
Aggressive OS guesses: Microsoft Windows Server 2008 R2 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-07-08T15:05:32
|_  start_date: 2024-07-08T15:03:20
|_clock-skew: mean: 4h59m59s, deviation: 0s, median: 4h59m59s

TRACEROUTE (using port 445/tcp)
HOP RTT       ADDRESS
1   479.87 ms 10.10.14.1
2   480.67 ms 10.10.10.63

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 143.80 seconds
</pre></td></tr></tbody></table></code></pre></div></div>

The http page on port 80 is just a fake askJeeves page and there's nothing to be found, even after dirbusting.

The next thing I checked was the 50000 port. The initial page looks like this:

![50000 page](/images/Jeeves50000page.png)

When I saw this, I immediately started dirbusting.

![dirbusting](/images/JeevesDirbusting.png)

The /askjeeves turned out to be a Jenkins page left without anything to stop me from using Groovy console whatsoever.
So, I used that to get my foothold

![Groovy](/images/JeevesGroovy.png)

The script that I used, was taken from [this](https://www.revshells.com/) awesome site.

I recently found cool shell handler called [penelope.py](https://github.com/brightio/penelope) and I think it's really usefull especially with linux shells since it automatically upgrades them to fully interactive ones!

![RevShell](/images/JeevesPenelope.png)

Script worked and I got my shell!

# PrivEsc

Quick enumeration led me to finding privilege "SeImpersonatePrivilege" set to enabled.

![whoami priv](/images/JeevesPrivs.png)

Right away, I thought of potato attack, so I quickly migrated to metasploit.

![Metasploit](/images/JeevesMetasploit.png)

I used the module called `exploit/local/ms16_075_reflection_juicy` and got the NT AUTHORITY\SYSTEM shell!

Unfortunately it's not the end of this box, finding the root flag was challenging as well.
To find the flag I had to use Alternate Data Streams, since the root.txt on Administrator's Desktop was fake.

To find the real flag you have to use command `dir /R` in the Administrator's Desktop, from there you can type it out using
`more < 'full file's path from dir /R'`

And that's all, thank you for reading!
