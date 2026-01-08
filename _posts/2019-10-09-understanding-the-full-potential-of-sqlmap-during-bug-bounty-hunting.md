---
layout: post
title: "Understanding the full potential of sqlmap during bug bounty hunting"
date: 2019-10-09 00:00:00 -0000
categories: ['Bug bounty', 'Ethical hacking', 'Tools']
tags: [sql injection, SQLi, sqlmap]
author: vavkamil
redirect_from:
  - /2019/10/09/understanding-the-full-potential-of-sqlmap-during-bug-bounty-hunting/
---

<hr>

<!-- wp:paragraph -->
<p>Swiss army knife for SQL Injection attacks, sqlmap was first developed in 2006 by Daniele Bellucci and later maintained by Bernardo Damele and Miroslav Stampar. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Its early development took off thanks to the <a rel="noreferrer noopener" aria-label="OWASP Spring of Code 2007 (opens in a new tab)" href="https://www.owasp.org/index.php/OWASP_Spring_Of_Code_2007_Applications#Bernardo_-_sqlmap" target="_blank">OWASP Spring of Code 2007</a> and was first under the serious media coverage during the <a rel="noreferrer noopener" aria-label="Black Hat Europe 2009 (opens in a new tab)" href="https://bernardodamele.blogspot.com/2009/03/black-hat-europe-2009.html" target="_blank">Black Hat Europe 2009</a> conference. If you are interested in more dates and early milestones, I highly recommend checking the <a rel="noreferrer noopener" aria-label="History (opens in a new tab)" href="https://github.com/sqlmapproject/sqlmap/wiki/History" target="_blank">History</a> page on the official <a rel="noreferrer noopener" aria-label="Wiki (opens in a new tab)" href="https://github.com/sqlmapproject/sqlmap/wiki" target="_blank">Wiki</a>. It's worth to mention that there is also a fairly recent <a rel="noreferrer noopener" aria-label="interview (opens in a new tab)" href="https://portswigger.net/daily-swig/sqlmap-plans-to-prototype-gui-support" target="_blank">interview</a> with Miroslav Stampar by The Daily Swig.</p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>Next-gen SQL injection opens server door; A vulnerability estimated to affect more than 1 in 10 websites could go lethal with the finding that it can be used to reliably take complete control of the site's underlying server.</p><cite><a href="https://www.theregister.co.uk/2009/04/02/new_sql_injection_attack/" target="_blank" rel="noreferrer noopener" aria-label="TheRegister.co.uk - Dan Goodin, 2 Apr 2009 (opens in a new tab)">TheRegister.co.uk - Dan Goodin, 2 Apr 2009</a></cite></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>Stable version 1.0 was released ten years later on Feb 27, 2016 and with continuous releases even to this day, sqlmap is number one tool for detecting and exploiting SQL injection flaws.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>So what exactly is sqlmap?</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p><em>sqlmap is an open source penetration testing tool that automates the process of detecting and exploiting SQL injection flaws and taking over of database servers. It comes with a powerful detection engine, many niche features for the ultimate penetration tester and a broad range of switches lasting from database fingerprinting, over data fetching from the database, to accessing the underlying file system and executing commands on the operating system via out-of-band connections.</em></p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4>Quick links:</h4>
<!-- /wp:heading -->

<!-- wp:list -->
<ul><li>Website: <a rel="noreferrer noopener" aria-label="http://sqlmap.org/ (opens in a new tab)" href="http://sqlmap.org/" target="_blank">http://sqlmap.org/</a></li><li>GitHub: <a rel="noreferrer noopener" aria-label="https://github.com/sqlmapproject/sqlmap (opens in a new tab)" href="https://github.com/sqlmapproject/sqlmap" target="_blank">https://github.com/sqlmapproject/sqlmap</a></li><li>Twitter: <a rel="noreferrer noopener" aria-label="https://twitter.com/sqlmap (opens in a new tab)" href="https://twitter.com/sqlmap" target="_blank">https://twitter.com/sqlmap</a></li><li>Wiki: <a rel="noreferrer noopener" aria-label="https://github.com/sqlmapproject/sqlmap/wiki (opens in a new tab)" href="https://github.com/sqlmapproject/sqlmap/wiki" target="_blank">https://github.com/sqlmapproject/sqlmap/wiki</a></li><li>YouTube: <a href="https://www.youtube.com/user/inquisb/videos" target="_blank" rel="noreferrer noopener" aria-label="https://www.youtube.com/user/inquisb/videos (opens in a new tab)">https://www.youtube.com/user/inquisb/videos</a></li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>I'm not by any means an expert on SQL Injection attacks, but I have been using this tool for several years and thought that it would nice to share some tips &amp; tricks with you. If you are in the web security field, it's necessary to understand the fundamentals of how SQLi works and everyone should be able to exploit this vulnerability manually. On the other hand, automation is the new trend :)</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The thing is, I saw numerous questions from the students/newbies in the bug bounty industry and if you are just blindly pasting URLs into the sqlmap, you are doing something wrong!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Sometimes people are passing the vulnerabilities that are there because they won't get the hit on the first try and just move on. I won't help you understand how SQL injection works, but this post can help you understand how to use the tool ... Hope this won't be super long in the end.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>If you ever read the <a rel="noreferrer noopener" aria-label="usage (opens in a new tab)" href="https://github.com/sqlmapproject/sqlmap/wiki/Usage" target="_blank">Usage</a> manual, there are so many options, switches, and features. I won't probably go over all of them, I will instead focus on the most common usage, some misconceptions, and recommendations. Overall, most of the readers are probably interested in bug bounty hunting and this is meant to be a guide for them, so if you are already experienced in the game, you can stop right here and save yourself some precious time :)</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>It's probably a good idea to preface this with an official sqlmap disclaimer: </p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program</p><cite>[!] legal disclaimer</cite></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>The first thing is to specify the URL for testing, you can do that with <code>-u</code> or <code>--url</code> parameter (<code>-r</code> for saved request):</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -u "http://0.0.0.0:1234/?id=2"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>By default, sqlmap performs HTTP requests with the user-agent like: <code>sqlmap/1.2.4#stable (http://sqlmap.org)</code>, which can get you blocked by some firewalls even before you start. So the next step is to change it to something more real. There are three options on how to do that, you can either specify custom user-agent header with <code><code>--user-agent</code></code>, use random one <code><code>--random-agent</code></code> or even imitate smartphone with <code><code>--mobile</code></code>.</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -u "http://0.0.0.0:1234/?id=2" --random-agent</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Protip: use SQLMAP_RANDOMAGENT environment variable to always use random user-agent:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ echo "export SQLMAP_RANDOMAGENT=1" >> ~/.bash_profile &amp;&amp; source ~/.bash_profile</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>You can do more tunning like this, just look into <code>/etc/sqlmap/sqlmap.conf</code> for some inspiration ...</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Even if some researchers are against it, there are bug bounty programs like Goldman Sachs or Verizon that will award you a bonus if you play nice and specify a custom identifiable header like <code>X-HackerOne-Research:username</code> or <code>X-Bug-Bounty:HackerOne-username</code>. You can do it easily with <code>--headers</code>:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -u "http://0.0.0.0:1234/?id=2" --headers="X-HackerOne:username"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>And most importantly, with a rising number of researchers and automated tools or scans, it is wise to follow the rules and limit your requests per second according to each program's policy. The last thing you want to do is to cause some infrastructure disturbances, outages or get yourself banned. If the program allows max 5 req/s, you can specify the <code><code>--delay</code></code> between each request to 200 milliseconds:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -u "http://0.0.0.0:1234/?id=2" --delay=0.2</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>With the info said above, there shouldn't be any reason to increase the number of <code>--threads</code> during bug bounty hunting unless you know what you are doing or don't care at all. There is also an option to create custom configuration files and use them with <code>-c</code> based on your needs, for example:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -u "http://0.0.0.0:1234/?id=2" -c ~/sqlmap.ini

$ cat ~/sqlmap.ini
&#91;Target]

&#91;Request]
delay = 0.2
randomAgent = True
headers = X-HackerOne: username </code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>This can be rather useful mainly while doing bug bounty hunting through own SOCKS5 proxy and don't want to configure it each time. Crafting the specific config according to your own needs will benefit you a lot in a long turn.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>If you are hunting for bugs only with Burp Suite Community Edition, you can also leverage some advantages to be able to effectively find SQLi vulnerabilities. Most of you probably already know about the Burp <a rel="noreferrer noopener" aria-label="SQLiPy extension (opens in a new tab)" href="https://support.portswigger.net/customer/portal/articles/2791040-using-burp-with-sqlmap" target="_blank">SQLiPy extension</a>, or open-source tools leveraging sqlmap API like <a rel="noreferrer noopener" aria-label="SQLiScanner (opens in a new tab)" href="https://github.com/0xbug/SQLiScanner" target="_blank">SQLiScanner</a> or <a href="https://github.com/zt2/sqli-hunter">SQLi-Hunter</a>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I would like to show you another useful feature. Let's say that you are browsing the target web application with Burp Suite and have a bunch of requests in Burp's HTTP history. You can select everything with CRTL+A, then continue with a right-click and "Save items" option. Just like this, it is possible to export the whole HTTP history as an XML file, for example: "burp_history.xml". With sqlmaps' <code>-l</code> option, just specify the log file:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -l burp_history.xml</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Don't forget that you should always stay in scope of the program, it can be done with <code>--scope</code> option (regex):</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -l burp_history.xml --scope="0.0.0.0:1234"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>With the examples above, it is easy to follow the rules of a bug bounty program and at least somehow automate the SQLi testing, but speaking about automation, it's not finished without the notifications. There is <code><code>--beep</code></code> switch which will make a sound when SQLi is found, but nowaday everything is running in the cloud, so this is not an option. Thankfully, there is another <code>--alert</code> option, which will run host OS command(s) when SQL injection is found. Basically in my scenario, it will run a simple bash script to send me a Telegram notification, this can be easily changed to webhook/slack/discord message ...</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The thing that was bothering me the most, if I'm running a lot of different scans, how the bash script can differentiate between them. For example, each scan can have unique identification ID, but it would be cool if the bash script could notify me with the whole sqlmap command with all the parameters used. After a lot of trial/error bash fu, I came up with the following solution:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -u "http://0.0.0.0:1234/?id=2" --alert="./sqli2telegram.sh $$"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>And the terrible bash script itself:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"right","id":665,"width":288,"height":512,"linkDestination":"media"} -->
<div class="wp-block-image"><figure class="alignright is-resized"><a href="{{ '/assets/img/2019/10/Screenshot_20191009-223715_Telegram.png' | relative_url }}"><img src="{{ '/assets/img/2019/10/Screenshot_20191009-223715_Telegram-576x1024.png' | relative_url }}" alt="" class="wp-image-665" width="288" height="512"/></a><figcaption>Telegram SQLi notification</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:code -->
<pre class="wp-block-code"><code>#!/bin/bash
command="$(ps -f | grep $@ | grep sqlmap | ps ww -o cmd= -p $(cut -d' ' -f 2))"
date="$(date "+%d %b %Y %H:%M")" #Collect date &amp; time.

text="&lt;b>SQLi found !&lt;/b>%0A
&lt;i>$date&lt;/i>%0A
&lt;code>$command&lt;/code>"

# Telegram:
user="***SECRET***"
key="***SECRET***"
url="https://api.telegram.org/bot$key/sendMessage"

curl -s --max-time 10 -d "chat_id=$user&amp;disable_web_page_preview=1&amp;parse_mode=html&amp;text=$text" $url > /dev/null</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>This is not the best solution at all and might cause some issues, but only your imagination is a limit, perhaps people will share any ideas on how to do things better. The whole command with all the mentioned examples might look like this:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sqlmap -c ~/sqlmap-config.ini -l ~/burp-history.xml --scope="0.0.0.0:1234" --batch --alert="~/sqli2telegram.sh $$"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The <code>--batch</code> option will tell the sqlmap to never ask for user input and always use the default behavior. There is a ton of other <a rel="noreferrer noopener" aria-label="useful features (opens in a new tab)" href="https://github.com/sqlmapproject/sqlmap/wiki/Usage" target="_blank">useful features</a>, it is always recommended to experiment and find what will suit you the best.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>For the testing, I highly recommend to use <a rel="noreferrer noopener" aria-label="DSVW (Damn Small Vulnerable Web) (opens in a new tab)" href="https://github.com/stamparm" target="_blank">DSVW (Damn Small Vulnerable Web)</a>, also authored by Miroslav Stampar :) By running <code>docker run -p 1234:8000 -it appsecco/dsvw</code> you will have end-points for all interesting vulnerabilities, including SQLi. I'm putting together a list of another <a rel="noreferrer noopener" aria-label="awesome vulnerable applications (opens in a new tab)" href="https://github.com/vavkamil/awesome-vulnerable-apps" target="_blank">awesome vulnerable applications</a>, but it's not done yet, help is always welcomed ...</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>While testing for SQLi, don't forget that every user input might be vulnerable. There might be a vulnerability in cookies, headers like referer, host etc. Quite recently on <a rel="noreferrer noopener" aria-label="/r/bugbounty (opens in a new tab)" href="https://pentestmag.com/exploiting-blind-sql-injections-update-insert-statements-without-stacked-queries-sina-yazdanmehr/" target="_blank">/r/bugbounty</a>, I released a CTF challenge to celebrate 5k subscribers. The solution was blind SQLi in insert (sqlite database), and the vulnerable parameter was user-agent. It took a bunch of researchers a lot of hours and tens/hundreds of thousands of  requests to find it. It could be found quite easily just with running sqlmap and increasing<code><code>--level</code></code> and <code>--risk</code> options and speed up fingerprinting and specifying the right database. Always try harder and look beyond the low hanging fruits!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>PS: I forgot to mention the <a rel="noreferrer noopener" aria-label="tamper scripts (opens in a new tab)" href="https://docs.google.com/spreadsheets/d/10nMVkPNi5VS2s4Ju40IrvvaTO-grO_BTT5gJBtEhrb0/edit?source=post_page-----91be42dfc893----------------------#gid=0" target="_blank">tamper scripts</a> :(</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>Resources:</strong></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>https://www.owasp.org/index.php/Automated_Audit_using_SQLMap<br> https://github.com/sqlmapproject/sqlmap<br> http://sqlmap.org/<br> https://github.com/sqlmapproject/sqlmap/wiki/FAQ<br>https://jlajara.gitlab.io/posts/2019/04/29/Second_order_sqli.html<br> https://github.com/sqlmapproject/sqlmap/wiki/Features<br> https://github.com/sqlmapproject/sqlmap/wiki/Usage<br> https://blog.haschek.at/2017/how-to-defend-your-website-with-zip-bombs.html<br> https://pentest.blog/data-ex-filtration-with-dns-in-sqli-attacks/<br> https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/sqlmap-tricks-for-advanced-sql-injection/<br> https://www.scanforsecurity.com/scanning-techniques/sqlmap-advanced-guide.html<br> http://malwrforensics.com/en/2018/07/07/sqlmap-advanced-tips-and-tricks/<br> https://miloserdov.org/?p=1403<br> https://web.archive.org/web/20150328010321/https://www.trustwave.com/Resources/SpiderLabs-Blog/Sqlmap-Tricks-for-Advanced-SQL-Injection/<br> https://twitter.com/sqlmap<br>https://pentestmag.com/exploiting-blind-sql-injections-update-insert-statements-without-stacked-queries-sina-yazdanmehr/<br> https://medium.com/@rrubymann/use-sqlmap-to-find-xss-vulnerabilities-afa8aa690510<br> https://github.com/RhinoSecurityLabs/SleuthQL<br> https://medium.com/@bugbsurveys/sqlmap-tamper-scripts-91be42dfc893<br> https://docs.google.com/spreadsheets/d/10nMVkPNi5VS2s4Ju40IrvvaTO-grO_BTT5gJBtEhrb0/edit?source=post_page-----91be42dfc893----------------------#gid=0</p>
<!-- /wp:paragraph -->