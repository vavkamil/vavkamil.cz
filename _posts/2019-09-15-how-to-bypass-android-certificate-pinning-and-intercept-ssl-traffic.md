---
layout: post
title: "How to bypass Android certificate pinning and intercept SSL traffic"
date: 2019-09-15 00:00:00 -0000
categories: ['Bug bounty', 'Ethical hacking']
tags: [android, burp suite, certificate pinning, frida, HTTPS, mitm, objection, proxy]
author: vavkamil
redirect_from:
  - /2019/09/15/how-to-bypass-android-certificate-pinning-and-intercept-ssl-traffic/
---

<hr>

<!-- wp:paragraph -->
<p>Over the last few months, I had a quite luck finding IDOR vulnerabilities in mobile API of Android applications. Nowadays most of the apps are obfuscated and using certificate pinning to prevent MiTMs.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>If you are late in the game or want to shift your bug bounty hunting on Android apps, there are awesome tools that can help you catch up fairly quickly :)</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I'm myself using an older rooted phone with Burp Suite and tools mentioned bellow.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>Prerequisites:</strong></p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>smartphone with <strong>rooted</strong> Android 7+ (I'm using Nexus 5x with latest LineageOS; Android 8.1.0)</li><li>computer with Linux (latest Ubuntu is just fine)</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p><strong>Requirements:</strong></p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li><a rel="noreferrer noopener" aria-label="Burp Suite (opens in a new tab)" href="https://portswigger.net/burp/communitydownload" target="_blank">Burp Suite</a> (community edition is fine)</li><li><a href="https://developer.android.com/studio/command-line/adb">ADB</a> (sudo apt-get install adb OR sudo yum install android-tools)</li><li><a rel="noreferrer noopener" aria-label="Frida (opens in a new tab)" href="https://github.com/frida/frida" target="_blank">Frida</a> (pip install frida-tools)</li><li><a href="https://github.com/sensepost/objection" target="_blank" rel="noreferrer noopener" aria-label="Objection (opens in a new tab)">Objection</a> (pip3 install objection) </li></ul>
<!-- /wp:list -->

<!-- wp:heading -->
<h2>Intercepting HTTPS traffic</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Since the Android Nougat 7.0 (API &gt;= 24), it is no longer possible to simply install the Burp Suite CA certificate, as it's no longer trust user or admin-added CAs for secure connections, by default.</p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>Protection of all application data is a key goal of the Android application sandbox. Android Nougat changes how applications interact with user- and admin-supplied CAs. By default, apps that target API level 24 will—by design—not honor such CAs unless the app explicitly opts in. This safe-by-default setting reduces application attack surface and encourages consistent handling of network and file-based application data.</p><cite>https://android-developers.googleblog.com/2016/07/changes-to-trusted-certificate.html</cite></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>For the guide on how to install Burp cert as a system with root  permissions, best description that I come over was from <a href="https://twitter.com/ropnop" target="_blank" rel="noreferrer noopener" aria-label="@ropnop (opens in a new tab)">@ropnop</a>, so please look at his article <a rel="noreferrer noopener" aria-label="Configuring Burp Suite with Android Nougat (opens in a new tab)" href="https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/" target="_blank">Configuring Burp Suite with Android Nougat</a>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Thanks to the <a href="https://twitter.com/securitychops" target="_blank" rel="noreferrer noopener" aria-label="@securitychops (opens in a new tab)">@securitychops</a>, we have <a rel="noreferrer noopener" aria-label="one liner (opens in a new tab)" href="https://securitychops.com/2019/08/31/dev/random/one-liner-to-install-burp-cacert-into-android.html" target="_blank">one liner</a> that will do everything for you :)</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"ad5ddbeec4685c6bca271063d0c95054"} /-->

<!-- wp:html -->
<p>Next, you need to <a rel="noreferrer noopener" aria-label="install (opens in a new tab)" href="https://www.frida.re/docs/installation/" target="_blank">install Frida framework</a> to your PC. The best way to install Frida’s CLI tools is via PyPI:</p>
<!-- /wp:html -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ pip3 install frida-tools</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Frida is like Greasemonkey for native apps, or, put in more technical terms, it’s a dynamic code instrumentation toolkit. It lets you inject snippets of JavaScript or your own library into native apps. Frida also provides you with some simple tools built on top of the Frida API. These can be used as-is, tweaked to your needs, or serve as examples of how to use the API.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>It also needs to run on the phone, so enable USB debugging and connect to your device. To install ADB &amp; Fastboot on Ubuntu systems, execute the following command from the terminal:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ sudo apt-get install android-tools-adb android-tools-fastboot</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>I prefer connection over the network  because sometimes I don't have USB-C cable laying around and this is a workaround most of the time.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"left","id":628,"width":270,"height":290} -->
<div class="wp-block-image"><figure class="alignleft is-resized"><img src="{{ '/assets/img/2019/09/android-debugging.png' | relative_url }}" alt="android debugging" class="wp-image-628" width="270" height="290"/></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>On the phone, allow debugging in <code>Settings/System/Developer options</code> and turn on `ADB over network`. You should be able to connect like this:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ adb connect 10.10.10.10:5555
* daemon not running; starting now at tcp:5037
* daemon started successfully
connected to 10.10.10.10:5555
$ adb root
restarting adbd as root
$ adb connect 10.10.10.10:5555
connected to 10.10.10.10:5555
$ adb shell</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Now you need to download frida-tools to your device and start the service. There is a <a rel="noreferrer noopener" aria-label="nice guide (opens in a new tab)" href="https://github.com/dpnishant/appmon/wiki/5.a-Setup-on-Android-%5BRooted%5D" target="_blank">nice guide</a> from <a href="https://twitter.com/dpnishant" target="_blank" rel="noreferrer noopener" aria-label="@dpnishant (opens in a new tab)">@dpnishant</a> how to do that:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ wget https://github.com/frida/frida/releases/download/12.7.0/frida-server-12.7.0-android-arm64.xz
$ unxz frida-server-12.7.0-android-arm64.xz
$ mv frida-server-12.7.0-android-arm64 frida-server
$ adb push frida-server /data/local/tmp/
$ adb shell "chmod 755 /data/local/tmp/frida-server
$ adb shell "/data/local/tmp/frida-server &amp;
</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Don't forget that root permissions on the phone are required. Next, start the installation of Objection using pip3 with:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ pip3 install objection</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Objection is a runtime exploration toolkit powered by Frida. There is an <a rel="noreferrer noopener" aria-label="awesome article (opens in a new tab)" href="https://sensepost.com/blog/2017/objection-mobile-runtime-exploration/" target="_blank">awesome article</a> from the author himself. It has a lot of <a href="https://github.com/sensepost/objection#features" target="_blank" rel="noreferrer noopener" aria-label="cool features (opens in a new tab)">cool features</a> for both Android and iOS.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>At this point, <strong>frida-server</strong> is running on the phone, you are connected over adb from the PC and have installed all prerequisites.<br>To be able to use a smartphone with Burp Suite, you need to change proxy listener from <code>Loopback only</code> to <code>All interfaces</code>.<br>And modify Wi-Fi on the phone. In the advanced options you can change proxy settings, so with the manual settings enter the local IP and port for Burp Suite.</p>
<!-- /wp:paragraph -->

<!-- wp:gallery {"ids":["634","633"]} -->
<figure class="wp-block-gallery cpngolumns-2 is-cropped"><ul class="blocks-gallery-grid"><li class="blocks-gallery-item"><figure><img src="{{ '/assets/img/2019/09/burp-interfaces.png' | relative_url }}" alt="" data-id="634" data-link="https://vavkamil.cz/?attachment_id=634" class="wp-image-634"/><figcaption class="blocks-gallery-item__caption">Burp - All interfaces</figcaption></figure></li><li class="blocks-gallery-item"><figure><img src="{{ '/assets/img/2019/09/android-wifi-proxy.png' | relative_url }}" alt="" data-id="633" data-link="https://vavkamil.cz/?attachment_id=633" class="wp-image-633"/><figcaption class="blocks-gallery-item__caption">AndroidWi-Fi proxy - Burp IP</figcaption></figure></li></ul></figure>
<!-- /wp:gallery -->

<!-- wp:paragraph -->
<p>Finally, its time to select any installed Android application and try to bypass certificate pinning and see requests in Burp suite. With Frida, you list application on the phone like this:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ frida-ps -Ua</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>Or if you already know the identifier of the application, for example <code>com.*******.kln</code>, hook it with Objection like this:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>$ objection -g c**********n explore -q</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>And the best part is that after running:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code># android sslpinning disable</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>you should be able to see all requests from a mobile application in Burp Suite with successfully bypassed certificate pinning.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":635,"width":495,"height":285} -->
<div class="wp-block-image"><figure class="aligncenter is-resized"><img src="{{ '/assets/img/2019/09/android-sslpinning-disabled.png' | relative_url }}" alt="" class="wp-image-635" width="495" height="285"/><figcaption>android sslpinning disable</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Sometimes if you experience some issues, for example with Facebooks authentication or things like that, it's best to configure "SSL Pass Through" in "Burp Suite &gt; Proxy &gt; Options" check <code>Automatically add entries on client SSL negotiation failure</code>.</p>
<!-- /wp:paragraph -->

<!-- wp:gistr/custom-block {"content":"91c77a6f32fb0eaac4498de662e2aa56"} /-->

<!-- wp:paragraph -->
<p>Back to the Burp proxy, I once did this entire setup and the first thing that I saw in HTTP history was this request:</p>
<!-- /wp:paragraph -->

<!-- wp:code -->
<pre class="wp-block-code"><code>GET /api/2.0/user/64296783669/messages HTTP/1.1
X-SessionId: d98d861ef5fd
X-Token: ak1tUXhPRFl5WkdZeU1qa3pOakZrT0RJMFpUazVaV1E1Wmc=
User-Agent: App|Android|2.0.0.0|2.3
Connection: close
Host: redacted
Accept-Encoding: gzip, deflate</code></pre>
<!-- /wp:code -->

<!-- wp:paragraph -->
<p>and just by modifying userID was able to see other user's messages :) That was a lucky day.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4><strong>Resources:</strong></h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p><em>https://www.frida.re/<br>https://www.frida.re/docs/installation/<br>https://support.google.com/nexus/answer/2844832?hl=en</em><br><em>https://sensepost.com/blog/2017/objection-mobile-runtime-exploration/<br>https://omespino.com/tutorial-universal-android-ssl-pinning-in-10-minutes-with-frida/<br>https://github.com/sensepost/objection/wiki/Installation<br>https://github.com/sensepost/objection<br>https://github.com/frida/frida/issues/597<br>https://github.com/dpnishant/appmon/wiki/5.a-Setup-on-Android-%5BRooted%5D<br>https://gist.github.com/davidnunez/1404789<br>https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/<br>https://blog.netspi.com/four-ways-bypass-android-ssl-verification-certificate-pinning/<br>https://blog.jamie.holdings/2018/09/05/bypass-certificate-pinning-on-android/</em><br><em>https://www.notsosecure.com/pentesting-android-apps-using-frida/</em></p>
<!-- /wp:paragraph -->