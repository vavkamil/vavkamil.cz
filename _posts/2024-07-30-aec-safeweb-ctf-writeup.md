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

### Level 4

- Url: https://safeweb.aec.cz/level4.php?uid=1
- Task: _Log in as the admin user_
- Hint: _SQL Injection_

Level #4 is the same login form but with a twist. The `uid` parameter fetches the username from the database and prefills the login input.

![ctf_04.png](/assets/img/2024/07/ctf_04.png)

We are looking at a classic SQL Injection here, and the best tool for this task is `sqlmap`. To do this, we can right-click on the GET request and use `Copy to file` in Burp. You can then use the request with `sqlmap` like this:
```
$ sqlmap --threads=10 -r sqli.txt --dbs
$ sqlmap --threads=10 -r sqli.txt --tables -D testdb_2014_1
$ sqlmap --threads=10 -r sqli.txt --columns -T users -D testdb_2014_1
$ sqlmap --threads=10 -r sqli.txt --dump -T users -D testdb_2014_1

Database: testdb_2014_1                                                                      
Table: users
[3 entries]
+----+-------+-------------+
| id | login | password    |
+----+-------+-------------+
| 1  | john  | Password    |
| 2  | bob   | abc123      |
| 0  | admin | asdfasdf123 |
+----+-------+-------------+
```

And just like that, we solved another level.

### Level 5

- Url: https://safeweb.aec.cz/level5.php
- Task: _Log in as the admin user. Your user credentials are: guest/tajneheslo_
- Hint: _Cookies_

This level is another login form, but now we have guest access. The goal is to escalate permissions to the admin using Cookies. After successful authentication, we can observe that a server response sets a `login` cookie.

![ctf_05.png](/assets/img/2024/07/ctf_05.png)

Send the authenticated GET request to Repeter, highlight the cookie value, and you will see that it is our `base64` encoded username. We can change the value to `admin`, resend the request, and solve the level.

![ctf_06.png](/assets/img/2024/07/ctf_06.png)

### Level 6

- Url: https://safeweb.aec.cz/level6.php
- Task: _Invoke an XSS error where you automatically call the alert() function displaying the contents of the cookie._
- Hint: _A multiline comment in JavaScript_

Level #6 is a funny reference to Czech hacking history, when `Mr.Sysel` proved that a blog can be hacked with a limited number of characters.

We have another login form, but it has four different inputs this time. By observing the form, each input has a limit of 30 characters, but the challenge is to execute:

`<script>alert(document.cookies)</script>`

which does have 40 characters. To verify which inputs are vulnerable to XSS, we will use these payloads:
- `">foo`
- `">bar`
- `">baz`
- `">qux`

![ctf_07.png](/assets/img/2024/07/ctf_07.png)

The first three inputs are vulnerable. Based on the hint, we have to use JS multiline comments. We will start the XSS payload in the first input, comment on the rest of the HTML code, end the comment in the second input, enter the payload, and comment on the rest. We will end the comment and the javascript payload in the last input.

It sounds way too complicated, but what we are doing is pretty much:

```javascript
"><script>/* commented HTML
commented HTML */ alert(document.cookie) /* commented HTML
commented HTML */</script>
```

![ctf_08.png](/assets/img/2024/07/ctf_08.png)


