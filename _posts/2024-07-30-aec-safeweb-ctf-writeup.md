---
layout: post
title: "AEC SafeWeb CTF Write-up"
date: 2024-07-30 00:00:00 -0000
categories: ['Ethical hacking', 'Security Research', 'Responsible disclosure']
tags: [ctf, writeup, hacking, security]
author: vavkamil
---

AEC Hacking Competition is a famous Czech CTF with 15 web application security challenges. It used to work like this: If you solved at least 12 challenges, you were invited for the penetration tester interview.

Today, only around 28 people out of several hundred could solve all the challenges. Many ethical hackers were stuck on some levels, maybe because the instructions were not clear or outdated.

The CTF is from ~2018, and the authors haven't updated any of the challenges since. We could do better, especially for new people starting in the penetration testing field. I was the sixth person who solved all the challenges, and I received an e-mail recently from someone stuck on the first level asking for advice.

So, after many years, I finally decided to try to solve them again and describe my steps. I did this mainly to help beginners see that it's easy if they think like ethical hackers and have the correct mindset and necessary skills. I would also like to see new levels that reflect the current application security state and the OWASP Top 10.

### Level 1

- Url: https://safeweb.aec.cz/level1.php
- Task: _Log in as an information system admin._
- Hint: _A cross between a Jack Russel terrier and a dachshund._

At the first level, we have a login form with `user:pass` inputs and a hint indicating a specific dog (which could be the password). We have to guess the username. The dog's name is `Krasty`. It was the mascot of the company Seznam and used to be the most famous dog in Czechia. Unfortunately, he "retired" many years ago, so chances are most young people won't get the idea.

To solve this, we can send the POST request to Burp's Intruder, with one payload set for `login` and the other for `password`. We will be using `Cluster bomb` attack with the following configuration:

- Payload set 1: Usernames (List)
- Payload set 2: Case modification (krasty)

![ctf_01.png](/assets/img/2024/07/ctf_01.png)

![ctf_02.png](/assets/img/2024/07/ctf_02.png)

After 9k requests, we can see that the correct credentials are `administrator:Krasty`, and we solved the first level.

### Level 2

- Url: https://safeweb.aec.cz/level2.php
- Task: _Log in as the admin user_
- Hint: _User-Agent_

Again, we have a login form with `user:pass` inputs, and based on the hint, we will have to modify the user agent. The form's title, `D-Link Router - Firmware DI-604UP,` indicated an exploit against an old Wi-Fi router. After a bit of googling, it looks like **CVE-2013-026**, where a backdoor in the firmware allowed to bypass authentication if one sets `xmlset_roodkcableoj28840ybtide` string as User-Agent.

![ctf_03.png](/assets/img/2024/07/ctf_03.png)

So, let's send the authentication request to Repeter and test it out. And just like that, we solved another level.

### Level 3

- Url: https://safeweb.aec.cz/level3.php
- Task: _Log in as the admin user_
- Hint: _There are backups of sensitive data on the server_

Level #3 is the same login form, and the hint indicates there is some backup file we can use to get the credentials. Observing the POST request in Burp, we can see the following path is used:
- `POST /library/login.php HTTP/1.1`

After navigating to:
- `https://safeweb.aec.cz/library/`

There are two files:
- login.php
- topsecret.txt
With the credentials:
```
Login: nbusr
Password: nbusr123
```

These credentials are an excellent reference to a time when our National Security Agency (NBU) was compromised because they were literally the credentials they were using at the time. And thanks to them, we just solved another level.