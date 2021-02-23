---
layout: post
title: "WP GDPR Compliance <= 1.5.5 - Unauthenticated Cross-Site Scripting (XSS)"
date: 2021-02-16 00:00:00 -0000
categories: ['Ethical hacking', 'Responsible disclosure', 'Bug bounty']
tags: [0day, exploit, vulnerability, WordPress]
author: vavkamil
---

tl;dr: The GDPR Compliance <= 1.5.5 plugin allowed unauthenticated users to exploit Stored Cross-Site Scripting (XSS) in the administration panel, which might lead to the privilege escalation. That was due to clients' IP Addresses reflected in the plugin's dashboard without being correctly validated or escaped.

<hr>

I have just recently joined a [Detectify](https://detectify.com/) crowdsource team, and I must say the platform is impressive. So I promised myself that I would spend some of the weekends looking for WordPress vulnerabilities to contribute with modules to the scanner. For the vulnerability to be accepted, the plugin must have at least 200k  installations.


I started browsing [popular](https://wordpress.org/plugins/browse/popular/) WP plugins, looking for ones that meet the criteria. After a while, I saw a GDPR plugin with 200,000+ active installations, and it caught my attention because I remember that there were some with critical vulnerabilities when the whole cookie consent thing was made mandatory and developers were racing with new plugins.

After checking the [plugin page](https://wordpress.org/plugins/wp-gdpr-compliance/), too see if there is any attack surface, one screenshot was interesting:

![](/assets/img/2021/02/gdpr-screenshot.png "Overview of the view and delete requests by your site's visitors.")

The description said: *Overview of the view and delete requests by your site's visitors.*, which indicated a dashboard in the admin panel with GDPR "delete requests" results, including the Email and IP Address of the user, could be a potential attack vector.

<hr>

### Discovery phase

After downloading the plugin and activating it in the [DVWP](https://github.com/vavkamil/dvwp) docker container, I published a page (with the form) to request deleting the user data and begin the black-box testing. Validation of the e-mail input was correct, but when I tried to spoof the IP address via `X-Forwarded-For: 1.1.1.1"><img src=x onerror=alert(1)>`, the XSS payload executed. What a surprise, it took me less than 10 minutes to find a "*Blind XSS*" vulnerability triggered in the context of a privileged user.

![](/assets/img/2021/02/gdpr-data-access-request.png "Data Access Requests")

<hr>

### Root cause analysis

At that point, I finished the testing, and I quickly moved to a source code review to locate the vulnerable code and continue with the white-box testing. Quick *grep* command revealed a database column `` `ip_address` varchar(255) NOT NULL ``, which was a nice surprise to see because thanks to that, it was possible to store the whole XSS payload. The IP address from the form is assigned being via `$request->setIpAddress(Helper::getClientIpAddress());` and the `getClientIpAddress()` is pretty much a standard function to check several headers for proxies and stuff like that. But what was confusing is that there was a `self::validateIpAddress($ipAddress)` call to validate the IP. The `validateIpAddress()` function:

```php
    /**
     * Ensures an ip address is both a valid IP and does not fall within
     * a private network range.
     *
     * @param string $ipAddress
     * @return bool
     */
    public static function validateIpAddress($ipAddress = '') {
        if (strtolower($ipAddress) === 'unknown') {
            return false;
        }
        // Generate ipv4 network address
        $ipAddress = ip2long($ipAddress);
        // If the ip is set and not equivalent to 255.255.255.255
        if ($ipAddress !== false && $ipAddress !== -1) {
            /**
             * Make sure to get unsigned long representation of ip
             * due to discrepancies between 32 and 64 bit OSes and
             * signed numbers (ints default to signed in PHP)
             */
            $ipAddress = sprintf('%u', $ipAddress);
            // Do private network range checking
            if ($ipAddress >= 0 && $ipAddress <= 50331647) return false;
            if ($ipAddress >= 167772160 && $ipAddress <= 184549375) return false;
            if ($ipAddress >= 2130706432 && $ipAddress <= 2147483647) return false;
            if ($ipAddress >= 2851995648 && $ipAddress <= 2852061183) return false;
            if ($ipAddress >= 2886729728 && $ipAddress <= 2887778303) return false;
            if ($ipAddress >= 3221225984 && $ipAddress <= 3221226239) return false;
            if ($ipAddress >= 3232235520 && $ipAddress <= 3232301055) return false;
            if ($ipAddress >= 4294967040) return false;
        }
        return true;
    }
```

After looking at the code for a longer time than I would like to admit, I realized that the whole logic is fundamentally flawed. The interesting part is the [ip2long](https://www.php.net/manual/en/function.ip2long.php) function, which generates a long integer representation of IPv4, which is then later checked via a list of known network ranges. But when the input is invalid, it will return **false**: `ip2long ( string $ip ) : int|false`. The `validateIpAddress()` is never catching that, so the valid IP is being validated, but the invalid IP will always return **true**, resulting in the payload stored in the database. And because the developer was confident in the check, it is never escaped when retrieved from the database and rendered in the admin dashboard.

<hr>

### Proof of Concept:

```
POST /wp-admin/admin-ajax.php HTTP/1.1
Host: 0.0.0.0:31337
X-Forwarded-For: 1.1.1.1"><img src=x onerror=alert(1)>

action=wpgdprc_process_action&security=cccf5a60ec&data={"type":"access_request","email":"xss@example.com","consent":true}
```

![](/assets/img/2021/02/gdpr-xss-poc.png "Proof of Concept")

<hr>

### Fix

Fix for the vulnerability [was released](https://plugins.trac.wordpress.org/changeset/2474862/)  in version 1.5.6 via adding to the check:

```
if ($ipAddress === false) {
    return false;
}
```

The user input is now correctly escaped, but the IP address column is still `varchar(255)`. Also, only the *PATCH* version was incremented, instead of the *MINOR* version, so it's hard to track the updated plugins' percentage via the advanced WordPress statistic. I believe it was a correct decision, but bumping it to `1.6` would be much better from the security point of view.

![](/assets/img/2021/02/gdpr-changelog.png "Changelog")

<hr>

When checking [WPScan](https://wpscan.com/vulnerability/678dac05-d1f7-4e73-a310-dffa8f5bb9c4) to verify that it's not a known vulnerability, I realized that this is, in fact, the (in)famous GDPR plugin, which resulted in a [full compromise](https://www.wordfence.com/blog/2018/11/privilege-escalation-flaw-in-wp-gdpr-compliance-plugin-exploited-in-the-wild/)  of hundreds/thousands of websites back in 2018. But I must say that I was impressed by the fast response & fix from the developer. Unfortunately for me, stored XSS is not a valid finding for Detectify. I did a quick recon for HackerOne in-scope items but didn't find any hit. Maybe I will be lucky next time :)

<hr>
By observing a spike in the "[Downloads per day](https://wordpress.org/plugins/wp-gdpr-compliance/advanced/)" graph during the week after the fix release, we can estimate that approximately ~50k websites updated the plugin, which is about 1/4 of all active installations. The stats are somewhat consistent with past releases and could indicate that we won't see any more websites updating to the latest version, so it might be a good idea to spread the news.

| Date       | Downloads |
| ---------- | --------- |
| 2021-02-15 | 23541     |
| 2021-02-16 | 12869     |
| 2021-02-17 | 5757      |
| 2021-02-18 | 3964      |
| 2021-02-19 | 3107      |
| 2021-02-20 | 2076      |
| 2021-02-21 | 1808      |
| 2021-02-22 | 3192      |

![](/assets/img/2021/02/gdpr-downloads.png "Downloads Per Day")

<hr>

### Timeline

* Friday, February 12th, 2021 ~ Notified a developer about the vulnerability
* Friday, February 12th, 2021 ~ Finished testing and proposed a fix
* Saturday, February 13th, 2021 ~ Received a response confirming that the developer received the information and will begin working on a fix
* Monday, February 15th, 2021 ~ The developer released a patched version of the plugin as of version 1.5.6
* Tuesday, February 23th, 2021 ~ Blog post with the details disclosed to the public
