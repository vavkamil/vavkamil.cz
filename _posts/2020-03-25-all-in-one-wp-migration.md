---
layout: post
title: "All-in-One WP Migration <=7.14 Arbitrary Backup Download"
date: 2020-03-25 00:00:00 -0000
categories: ['Ethical hacking', 'Responsible disclosure']
tags: [0day, exploit, vulnerability, WordPress]
author: vavkamil
---

<hr>

<!-- wp:paragraph -->
<p>A long time ago, I made a stupid decision to use WordPress for this blog about offensive website security. Since then, I learned a lot. I will be releasing a plugin to defend against <a rel="noreferrer noopener" aria-label="XML-RPC attacks (opens in a new tab)" href="https://twitter.com/vavkamil/status/1231298635561930752" target="_blank">XML-RPC attacks</a> and guide how to generate a static HTML site in upcoming weeks.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>But today I would like to share an interesting vulnerability that I found in a popular WordPress plugin with 2+ million active installations. I was looking for an easy-to-use backup plugin and <a href="https://wordpress.org/plugins/all-in-one-wp-migration" target="_blank" rel="noreferrer noopener" aria-label="All-in-One WP Migration (opens in a new tab)">All-in-One WP Migration</a> by ServMask seemed like a good choice.</p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>This plugin exports your WordPress website including the database, media files, plugins and themes with no technical knowledge required. Move, transfer, copy, migrate, and backup a site with 1-click. Quick, easy, and reliable.</p><cite>All-in-One WP Migration plugin description</cite></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>After installing the plugin and creating the first backup, the filename didn't look too random. I was wondering if it could be guessed by an attacker and somehow downloaded without the authenticated access.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":770,"sizeSlug":"large"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="{{ '/assets/img/2020/03/backup-1024x179.png' | relative_url }}" alt="" class="wp-image-770"/><figcaption>vavkamil.cz-20200324-214633-123.wpress</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>While checking the code, I  found out that it's just a <strong>domain-date-time-rand(100,999).wpress</strong> and the backup itself was publicly accessible via <strong>vavkamil.cz/wp-content/ai1wm-backups/vavkamil.cz-20200324-214633-123.wpress</strong></p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":771,"sizeSlug":"large"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="{{ '/assets/img/2020/03/backup-source.png' | relative_url }}" alt="" class="wp-image-771"/><figcaption>Function to generate file name</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>This seemed bad, but it would still require a huge amount of requests and pure luck to brute-force it. In the "<strong>wp-content/ai1wm-backups</strong>" folder, there were three additional files preventing directory listing:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>index.php</li><li>.htaccess</li><li>web.config</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>Back then, I didn't know what a "<strong>web.config</strong>" is for, but I was able to download it. There was nothing useful, it's basically .htaccess for Microsoft IIS server. Normally I would probably stop there, but I just finished reading <a href="https://www.goodreads.com/book/show/46223297-permanent-record" target="_blank" rel="noreferrer noopener" aria-label="Permanent Record (opens in a new tab)">Permanent Record</a> by Edward Snowden and remembered about the metadata :)</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>By downloading the <strong>web.config</strong> file and checking the <strong>Last-Modified</strong> metadata header, we can determine the exact date a time when the plugin was installed. Let's assume the user will install the All-in-One WP Migration plugin and create a backup within the first 10 minutes after the installation. The attacker can brute-force the backup file name with approximately 600k requests.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I quickly wrote a <a rel="noreferrer noopener" aria-label="simple python script (opens in a new tab)" href="https://gist.github.com/vavkamil/b54683430bd21d7fdade2ebdcf70cc82" target="_blank">simple python script</a> to extract the metadata and used <a href="https://github.com/xmendez/wfuzz" target="_blank" rel="noreferrer noopener" aria-label="wfuzz (opens in a new tab)">wfuzz</a> as a proof of concept:</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>wfuzz -c -z range,33-59 -z range,100-999 -X HEAD --sc 200 https://vavkamil.cz/wp-content/ai1wm-backups/vavkamil.cz-20200324-2146FUZZ-FUZ2Z.wpress</strong></p>
<!-- /wp:paragraph -->

<!-- wp:image {"align":"center","id":772,"sizeSlug":"large"} -->
<div class="wp-block-image"><figure class="aligncenter size-large"><img src="{{ '/assets/img/2020/03/wfuzz.png' | relative_url }}" alt="" class="wp-image-772"/><figcaption>Proof of concept using Wfuzz</figcaption></figure></div>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>The wfuzz parameters can be slightly modified to add a reasonable delay between the installation and first backup creation, or you can just try pure luck.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Now when the attacker can determine the backup file name and download it, he can use <a href="https://github.com/fifthsegment/Wpress-Extractor" target="_blank" rel="noreferrer noopener" aria-label=".wpress extractor (opens in a new tab)">.wpress extractor</a> to dump the database content from the archive, or just install the plugin to a fresh website and import the backup.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>It's worth mentioning that even if you uninstalled the plugin, the "<strong>ai1wm-backups</strong>" folder with all the backups was not deleted, and therefore you may still be vulnerable.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I scanned <a rel="noreferrer noopener" aria-label="hackerone/bugcrowd scope (opens in a new tab)" href="https://github.com/arkadiyt/bounty-targets-data" target="_blank">hackerone/bugcrowd scope</a> for bug bounty hunting purposes and was able to find ~20 blogs with the plugin installed, but I wasn't able to exploit any (mainly as I didn't want to bother them with a huge amount of requests). On the other hand, I exploited a hand-full of websites during the responsible disclosure among my clients, and this blog was also vulnerable/exploitable.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I reported the issue to the author and it was quickly fixed in version 7.15 (Exclude web.config and .htaccess direct access from each other). That means that it's no longer possible to download .htaccess file from IIS and web.config file from Apache. With the next updates, there were introduced some changes and the backup file name is more random.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>Timeline:</strong></p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Jan 13, 2020: Reported the vulnerability</li><li>Jan 13, 2020: Response from vendor</li><li>Jan 15, 2020: Ticket marked as resolved</li><li>Jan 20, 2020: Version 7.15 released, vulnerability fixed</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p><strong>Credits</strong>: <em><a href="https://ixia.keysight.com/company/blog/missing-updates-and-site-misconfiguration-can-lead-exposed-backups" target="_blank" rel="noreferrer noopener" aria-label="https://ixia.keysight.com/company/blog/missing-updates-and-site-misconfiguration-can-lead-exposed-backups (opens in a new tab)">https://ixia.keysight.com/company/blog/missing-updates-and-site-misconfiguration-can-lead-exposed-backups</a></em></p>
<!-- /wp:paragraph -->