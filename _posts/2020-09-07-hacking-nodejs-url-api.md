---
layout: post
title: "Hacking Node.js legacy URL API"
date: 2020-09-07 00:00:00 -0000
categories: ['Ethical hacking', 'Security research', 'Bug bounty']
tags: [hacking, nodejs, security, frontend, bugbounty]
author: vavkamil
image: "/assets/img/posts/2020-09-07-hacking-nodejs-url-api.png"
---

Our security engineers ensure the highest possible safety of our services. Their weapon of choice? Penetration testing. It is a simulation of a cyber-attack conducted by authorized developers to discover and penetrate any security vulnerabilities in the system/app/service the team is developing. Other proactive steps include secure code review, dependency scanning, SAST, and asset monitoring.

## Introduction

### How to bypass hostname verification to exploit allowlist functions

> Originaly published at [code.kiwi.com/articles/hacking-node-js-legacy-url-api](https://code.kiwi.com/articles/hacking-node-js-legacy-url-api/)

Security is essential to us in Kiwi.com as it ensures our customers’ safety. We need to be proactive and search for possible security issues that might affect our business and potentially cause a loss of customers’ trust.

<br>

Sometimes the security issue can be discovered even in a service we use often. Node.js sits behind most of our frontends, interacting with other backends, and recently a very interesting vulnerability was found just within its URL API. Node.js is a very vast project, which unfortunately comes in hand with some of its parts turning into a legacy. If the issue hasn’t been encountered, it would have caused serious security problems. Attackers could have exploited how Node.js processes hostname, making it easy for them to bypass security checks and obtain access to the customer’s accounts. Luckily, our application security team discovered the issue before it could cause us any harm.

<br>

In this article, you will learn how our security engineers found these particular security issues and how they mitigated them to ensure as high security for our customers as possible.

Let me share the findings with you.

<hr>
<br>

## A small primer on Node.js’s URL parsing APIs

![foo](/assets/img/2020/09/01.png)

- `https://url.spec.whatwg.org/#concept-basic-url-parser`

First of all, we should get familiar with URL parsing APIs in Node.js. Looking at the currently available docs (Node.js v13.8.0 Documentation), we can see that the URL module provides utilities for URL resolution and parsing. It can be accessed using:

```
const url = require(‘url’);
```

<blockquote>
“The URL module provides two APIs for working with URLs: a legacy API that is Node.js specific, and a newer API that implements the same WHATWG URL Standard used by web browsers”, as stated in the docs.
</blockquote>

URL parsing is frequently used in allowlist/denylist functions to prevent various attack vectors. For example, checking the hostname of a given URL against a list of allowed origins, to prevent for example Open Redirect vulnerabilities.

<br>

Using Node.js’ URL module, you can use url.host and url.hostname functions, the main difference is that url.hostname does not include the port.

<hr>
<br>

## Usage

![foo](/assets/img/2020/09/02.png)

- `https://nodejs.org/docs/latest/api/url.html#url_url_hostname`

But there is also Legacy URL API, deprecated since v11.0.0:

The legacy urlObject (require(‘url’).Url) is created and returned by the url.parse() function.

![foo](/assets/img/2020/09/03.png)

- `https://nodejs.org/docs/latest/api/url.html#url_legacy_url_api`

If we compare both WHATWG and Legacy URL API, functionality, and results from the security point of view should be the same, but what could go wrong?

<hr>
<br>

## Allowed domains — testing methodology

We have implemented various server-side security checks. One of them is a function for allowed domains: requests from the frontend can only reach API endpoints on a server with an allowed hostname, in our case anything hosted on the `*.kiwi.com`:

![foo](/assets/img/2020/09/04.webp)

- `A snippet of allowlist function for allowed hostnames`

Even though your own security precautions can be solid, the vulnerability can be in the tools that you use. The Node.js URL API was used for the allowlist function, and it turned out that even if you do everything according to the documentation, you can still be vulnerable.

<br>

Allowlist/denylist functions are usually an interesting scope for penetration testers, as there are a lot of things that could go wrong. Bypassing such security checks could often lead to SSRF or open redirect vulnerabilities.

<br>

The workflow of testing such attack vectors might look like this:

Let’s say you have a parameter named “allowed-subdomain”, when passing any URL into it, the function will check the hostname and verify if the domain or subdomain is allowed to communicate with kiwi.com services.

`https://www.kiwi.com/?allowed-subdomain=test.kiwi.com`

- Allowlist check passed, communication with `test.kiwi.com` allowed

`https://www.kiwi.com/?allowed-subdomain=attacker.com`

- Allowlist check refused, communication with `attacker.com` denied

Some of the steps that attack might try to bypass the check includes changing URL schema, prefix, infix, and suffix to bypass allowlist.

```
https://www.kiwi.com/?allowed-subdomain=test.kiwi.com
https://www.kiwi.com/?allowed-subdomain=attacker.kiwi.com
https://www.kiwi.com/?allowed-subdomain=xkiwi.com.
https://www.kiwi.com/?allowed-subdomain=foo-kiwi.com
https://www.kiwi.com/?allowed-subdomain=kiwi-foo.com
https://www.kiwi.com/?allowed-subdomain=test.kiwi.com.attacker.com
https://www.kiwi.com/?allowed-subdomain=kiwi.com@attacker.com
```

None of the above worked, but during bug bounty/penetration testing, persistence is the key. You can even automate the testing with some tool and try fuzzing it with a list of most common bypass payloads. There is a lot of possibilities, to have some idea, you can check the following links:


- [swisskyrepo/PayloadsAllTheThings/SSRF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [swisskyrepo/PayloadsAllTheThings/CRLF](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
- [swisskyrepo/PayloadsAllTheThings/Redirects](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Open%20Redirect)

I would also highly recommend checking “[A New Era of SSRF -Exploiting URL Parser in Trending Programming Languages!](https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf)” talk from Orange Tsai, which was presented on the Black Hat USA 2017 conference.

<hr>
<br>

## What about CRLF Injection?

The term CRLF refers to Carriage Return (ASCII 13, \r) Line Feed (ASCII 10, \n). They’re used to note the termination of a line. A CRLF Injection attack occurs when a user manages to submit a CRLF into an application.

Appending `%0A%0D (\r\n)` after the domain name:

- `https://www.kiwi.com/?allowed-subdomain=kiwi.com%0A%0Devil.com`

Gotcha! We tricked the allowlist function and a domain “kiwi.comevil.com” was returned as hostname with access allowed. Attackers with possession of the “comevil.com” domain could use this for malicious purposes.

<hr>
<br>

## Impact — what could go wrong in Kiwi.com

This was easy to exploit the vulnerability with a significant impact on our users. An attacker could use the knowledge of such allowlist bypass in various ways. One of them was tricking the debug feature to override the GraphQL server address used for authentication.

<br>

For the victim, the functionality of Kiwi.com changed dramatically, as any of the future requests from the browser would go to the malicious server. This means that when the victim tried to log-in into his/her account, the credentials were sent to the attackers’ controlled domain (kiwi.comevil.com), instead of the official Kiwi.com GraphQL server.

<hr>
<br>

## Secure code review — root cause discovery

This was a white-box penetration test. Bypassing the allowlist function was an easy part, now we had to find a vulnerable code, determine the root cause, and propose a mitigation/fix.

<br>

With other members of the AppSec team, we quickly determined that the allowlist function is written correctly, but the validation problem was underlying in the Node.js module.

![foo](/assets/img/2020/09/05.webp)

- `Proof of Concept code`

As you can see in the screenshot above, the legacy URL API is vulnerable, but the new API is correctly parsing the hostname as it should be, based on the RFC.

At the time, we were using the core url.parse() method to verify the hostname. We looked up the [documentation](https://nodejs.org/docs/latest/api/url.html#url_legacy_url_api) and learned that the URL API is legacy and is deprecated.

But we weren’t able to find any security advisory warning about the CRLF vulnerability in a hostname parser.

<br>

We weren’t exactly sure if this is a 0day exploit or not, so I reported it to the Node.js bug bounty program just to be sure. https://hackerone.com/reports/771596 Interesting thing is that another member of our AppSec team found a vulnerability in the same module a year ago: https://hackerone.com/reports/395845.

<br>

The Node.js team responded that there are known security issues in URL API and as it’s considered legacy now, they are not planning to fix them. Fair enough, but it’s reasonable to assume that a lot of companies are still using it and there should be more security warnings.

<hr>
<br>

## Conclusion — stay safe out there

Best effort URL-parsing libraries are not a good choice for security controls, it is highly recommended to add additional checks to be sure that critical functionality is properly hardened.

<br>

Based on our findings we could conclude that once you are using Node.js, you should do some static code analysis/dependency scanning (ESLint) to determine if your codebase is relying on the legacy URL API.

<strong>Example of ESLint rule:</strong>

```javascript
"rules": {
    // Forbid import of legacy URL API
    "no-restricted-imports": [
      "error",
      {
        "name": "url",
        "message": "Please use built-in URL",
      },
    ],
},
```

_At Kiwi.com we are using “The Zoo” service registry for all our Gitlab repositories. It is an open-source project developed by our engineers who are always looking for the best available solutions and being proactive in the sense of innovation. This tool allows us to write a quick check, scan all our projects/repositories, and automatically create issues with the description for our developers or even the merge requests with the correct patch._

<br>

We also found some repositories from top companies on GitHub with the same or similar issues, so we are in the process of writing a CodeQL query to find most of them and to properly notify them about the potential security vulnerabilities.
