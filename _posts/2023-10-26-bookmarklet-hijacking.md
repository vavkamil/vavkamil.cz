---
layout: post
title: "Bookmarklet hijacking"
date: 2023-10-26 00:00:00 -0000
categories: ['Ethical hacking', 'Tools', 'Security Research']
tags: [xss, javascript, bookmarklet, hijacking]
author: vavkamil
---

I noticed an exciting link shared on Hacker News titled "[Wait, what's a bookmarklet?](https://news.ycombinator.com/item?id=38014069)".

I have been using them for a long time, and while reading that discussion, it was nice to see people sharing useful ones they use to make their everyday tasks easier.

The security engineering mind got me thinking about the security implications of saving bookmarks with JavaScript from other websites, especially when readers share links to the GitHub gists.

We already know executing any non-trusted JavaScript is a wrong move. People were being tricked into pasting stuff into the browser console on popular websites and got hacked that way. There were even massive phishing campaigns using malicious [bookmarklets against Discord users](https://breakdev.org/hacked-discord-bookmarklet-attacks/).

But what if the victim is cautious and examines the content of bookmarklet JS before saving them to bookmarks? People often hover over links to see if they lead to malicious sites.

So, I was thinking about changing the content before the user saved it to bookmarks and was experimenting with the ondragstart Event. I came up with a very simple Proof of Concept for Chromium-based browsers and Firefox - [https://gist.github.com/vavkamil/0b167814cabf8787cd4c4ab629614c6e](https://gist.github.com/vavkamil/0b167814cabf8787cd4c4ab629614c6e).

It's a simple PoC where the href attribute specifies the link's destination:
`javascript: (() => { alert(1); })();`

And after saved to bookmarks, it changes to:
`javascript: (() => { alert(2); })();`

You can see it hee: [https://xss.vavkamil.cz/bookmarklet.html](https://xss.vavkamil.cz/bookmarklet.html)

### Chromium Version 118.0.5993.88

<video src="/assets/img/2023/10/bookmarklets_chromium.webm"></video>

### Mozilla Firefox 119.0

<video src="/assets/img/2023/10/bookmarklets_firefox.webm"></video>
