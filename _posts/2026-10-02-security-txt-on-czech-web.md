---
layout: post
title: "security.txt on the Czech web: Scanning 1k popular .cz domains"
date: 2026-10-02 00:00:00 -0000
categories: ['Security research', 'Responsible disclosure']
tags: [writeup, research, security-txt, czechia, rfc9116]
author: vavkamil
image: "/assets/img/posts/2026-10-02-security-txt-on-czech-web.png"
---

Adding security.txt to the web should be easy. The RFC is fairly simple, and there aren't many ways to fail. Well, at least I thought that, until now. Let's look at how a very small standard can fail in surprisingly creative ways.

---

### Table of contents

- [The boring stuff](#the-boring-stuff)
- [The funny stuff](#the-funny-stuff)
- [Conclusion](#conclusion)

TL;DR adoption is worse than expected, scanner is here `https://github.com/vavkamil/rfc9116-check`

---

I have been working on a side project, which now involves parsing and validating security.txt. I have been using the file and promoting its adoption for a long time. And I finally decided to add it to this blog as well.

It's fairly simple, right? Just one .txt file with an email contact for responsible disclosure.

So I added the file, built the blog on GitHub Pages, and checked the result. To my surprise, there was a cached 404 page. No problem; the cache will hopefully expire, and everything will be good. I checked another page I host on GH pages and where I added security.txt a long time ago and surprise, the same issue.

It took me a while to figure out that Jekyll excludes dot-prefixed directories by default. To fix that, I added this to my `_config.yml`:

```
include:
 - .well-known
```

And for the other domains where I serve just static content, all that was needed was adding a blank `docs/.nojekyll` file to disable Jekyll processing. Worked great!

---

After that, I also updated my existing security.txt on all projects, according to the specs. But from some preliminary testing, I knew that nobody reads RFCs anymore, and for my new project I will have to expect that.

Now I live in Czechia, and that is my main audience. So naturally, I decided to scan the top .cz websites for QA :)

Unfortunately CZ.NIC does not publish the `.cz` zone file for unrestricted public download, and Alexa Top Websites is long dead.

For my sample, I downloaded the Tranco top 1M list which included 5,324 .cz domains. To make my life easier, I selected the first 1k for the research.

---

With the help of `GPT-5.6-Sol high`, I created a simple script to scan for `/.well-known/security.txt` files and classify them based on strict RFC validation errors.

> https://github.com/vavkamil/rfc9116-check

Now lets talk [RFC 9116](https://www.rfc-editor.org/info/rfc9116/) and [securitytxt.org](https://securitytxt.org/) specs/guidance:

- The file is available at `https://domain/.well-known/security.txt`.
- HTTPS is preserved across redirects.
- The response is `text/plain` with `charset=utf-8`.
- The content is valid UTF-8 and uses valid line endings, including the final newline.
- At least one `Contact` field contains a valid URI, such as `mailto:`, `tel:` or `https:`.
- Exactly one `Expires` field contains a valid, non-expired RFC 3339 timestamp.
- Expiry is not more than one year ahead, as recommended by the RFC.
- URI-based fields use valid and secure addresses.
- `Preferred-Languages`, when present, contains valid language tags and appears only once.
- `Canonical`, when present, matches the location from which the file was retrieved.
- OpenPGP cleartext signatures are structurally valid when present.
- Referenced `/.well-known/` resources and extension fields are registered with IANA.

> Some of these are strict requirements, while others, such as a long expiry, an off-domain redirect, or a Canonical mismatch, are reported only as warnings.

Well, in theory, all you need to have is [this](https://vavkamil.cz/.well-known/security.txt):

```
Canonical: https://vavkamil.cz/.well-known/security.txt
Contact: mailto:vavkamil@protonmail.com
Expires: 2027-01-13T10:00:00Z
Preferred-Languages: en, cs

```

and all is great :) It's not a rocket science.

---
<br>

## The boring stuff

So I started scanning the top 1k .cz domains and prayed for a great result. I mean, it's 2026, and responsible disclosure is all people are talking about nowadays.

The result is not what I expected :(

| Result               | Count           | Percentage |
|----------------------|----------------:|-----------:|
| Domains scanned      | 1,000           | 100.00%    |
| DNS resolved         | 973 / 1,000     | 97.30%     |
| security.txt found   | 161 / 1,000     | 16.10%     |
| RFC 9116 valid       | 20 / 161 found  | 12.42%     |
| RFC 9116 invalid     | 141 / 161 found | 87.58%     |
| Absent / fallback    | 640 / 1,000     | 64.00%     |
| Blocked (HTTP 403)   | 65 / 1,000      | 6.50%      |
| Unreachable          | 100 / 1,000     | 10.00%     |
| Other inconclusive   | 34 / 1,000      | 3.40%      |

> Among the top 1,000 Tranco-ranked .cz domains, 16% published a detectable security.txt file, and 12% of those passed strict RFC 9116 validation.

Not great, not terrible. Only 20 websites out of the top 1k are doing a really great job. But remember that we are doing strict validation. To understand what is going on, I will have to manually triage the results.

Looking at the error distribution, the main offender is the UTF-8 charset header. Of the 161 files found, 49.69% failed that check. Even the top ranked Google's file:

```curl
curl -I https://www.google.cz/.well-known/security.txt

HTTP/2 200 
content-type: text/plain
```

> It MUST have a Content-Type of "text/plain" with the default charset parameter set to "utf-8" (as per Section 4.1.3 of [RFC2046]).

What we want to see is `content-type: text/plain; charset=UTF-8`.

I scanned all of my projects:

```
$ python scan_security_txt.py mine.txt

#  Domain                HTTP  security.txt  RFC      Issue / note
------------------------------------------------------------------
1  cvealert.io            200  found         invalid  charset header
2  vavkamil.cz            200  found         valid
3  appsecaudit.cz         200  found         valid
4  proofrequired.io       200  found         valid
5  kybervizitka.cz        200  found         invalid  charset header
```

Interestingly, all the ones hosted as static GitHub pages passed the check, and the ones deployed as Cloudflare workers failed. Cloudflare determines MIME type from the `.txt` extension and returns `text/plain`, but evidently does not append `charset=utf-8`.

Fixing that is as easy as reading [docs](https://developers.cloudflare.com/workers/static-assets/headers/#custom-headers) and adding:

```
/.well-known/security.txt
  Content-Type: text/plain; charset=utf-8
```

to the `./public/_headers` file.

Anyways, for the sake of the analysis, if we ignore the charset check, this adds 14 files that failed solely because of the charset header. Our new total is 34 now!

The remaining **127 of 161 (78.88%)** would still be invalid.

---

The second most common problem is the lack of a required Expires field.

Which is somewhat understandable, as that requirement was added later. And nobody is checking RFCs for updates. It's hard enough to convince frontend developers to add one txt file in the first place, you expect them to keep updating that?

But yeah, another 66 domains (40.99%) of the total found failed that. This one shouldn't be ignored.

The remaining errors and warnings need a manual deep dive to better understand where the implementation went wrong. If you are interested in the distribution, here it is:

| Finding                      | Kind    | Domains | % found |
|------------------------------|---------|--------:|--------:|
| Charset header               | error   |      80 |  49.69% |
| No expiry                    | error   |      66 |  40.99% |
| Expiry over one year         | warning |      42 |  26.09% |
| Unregistered well-known URI  | error   |      33 |  20.50% |
| No final newline             | error   |      29 |  18.01% |
| Canonical mismatch           | warning |      26 |  16.15% |
| Contact missing mailto       | error   |      23 |  14.29% |
| Expired                      | error   |      17 |  10.56% |
| PGP signature present        | info    |      10 |   6.21% |
| Off-domain redirect          | warning |       9 |   5.59% |
| Invalid language tag         | error   |       8 |   4.97% |
| URI                          | error   |       8 |   4.97% |
| Syntax                       | error   |       5 |   3.11% |
| No contact                   | error   |       4 |   2.48% |
| HTTP redirect                | error   |       3 |   1.86% |
| MIME type                    | error   |       3 |   1.86% |
| UTF-8                        | error   |       3 |   1.86% |
| Expiry format                | error   |       2 |   1.24% |
| Duplicate languages          | error   |       1 |   0.62% |
| HTTPS                        | error   |       1 |   0.62% |
| Insecure URI                 | error   |       1 |   0.62% |
| Invalid content encoding     | warning |       1 |   0.62% |

---
<br>

## The funny stuff

RFC 9116 doesn't require websites to allow every automated client. However, it defines `security.txt` as a machine-parsable resource at a predictable location specifically designed for automated discovery.

In my scan, 65/1000 (6.50%) of my discovery attempts were blocked by WAF.

Now, if your security team added a `security.txt` file but hid it behind a firewall and a CAPTCHA, that's very counterproductive. I didn't bother to look at them at all, so these are excluded from the statistics.

---

### Expiry is the biggest semantic failure

First, I want to see how many expired records we can find.

```
jq -r '
  .[]
  | select(.flags | index("expired"))
  | .domain
' results-tranco-cz-top-1000/results.json

webzdarma.cz       2023-02-09  1331 days ago
oleje.cz           2023-12-31  1006 days ago
casablanca.cz      2024-01-01  1005 days ago
ambis.cz           2025-01-01  639 days ago
embedit.cz         2025-01-01  639 days ago
regzone.cz         2025-03-30  551 days ago
inpage.cz          2025-03-30  551 days ago
zonercloud.cz      2025-03-30  551 days ago
notino.cz          2025-04-16  534 days ago
mp.cz              2025-06-01  488 days ago
turris.cz          2025-11-06  330 days ago
brno.cz            2025-12-30  276 days ago
mironet.cz         2025-12-31  275 days ago
jobs.cz            2026-05-07  148 days ago
zalando.cz         2026-06-01  123 days ago
zalando-lounge.cz  2026-06-01  123 days ago
autoscout24.cz     2026-06-30  94 days ago
```

There are some notable examples, like Czech companies with great security teams, Turris, which is operated by the CZ domain registry, or Brno, the city where I live :)

Next, I wanted to see if some went crazy with the Expiry, as the RFC says:

> It is RECOMMENDED that the value of this field be less than a year into the future to avoid staleness.

```
jq -r '
  [.[] | select(.flags | index("long_expiry"))]
  | sort_by(.expires_at)
  | reverse
  | .[:10][]
  | [.domain, .expires_at[0:10]]
  | @tsv
' results-tranco-cz-top-1000/results.json

csas.cz          2099-12-31   ~73 years ahead
rb.cz            2050-01-01   ~23 years ahead
postsignum.cz    2040-11-17   ~14 years ahead
spa.cz           2038-01-01   ~11 years ahead
kb.cz            2030-12-31   ~4 years ahead
...
```

These most extreme examples are pretty much all Czech banks. It's nice to see they are already committed to security and are planning that far ahead :)

Publishing security.txt once is easy; keeping it current is the actual operational challenge.

`jewishmuseum.cz` publishes `Expires: 2027-01-01T00:00:00T00:00:00Z` - The time component appears twice.

`vinted.cz` publishes `Thu, 15 Oct 2026 17:30:53 -0000` - email-style date rather than RFC 3339.

Both deal with history and past stuff, so one would expect that they will get the dates right :)

Across all 161 files:

| Expiry state                         | Domains | % found |
|--------------------------------------|---:|--------:|
| Present, current, under one year     | 31 | 19\.25% |
| More than one year ahead             | 42 | 26\.09% |
| Already expired                      | 17 | 10\.56% |
| Malformed                            | 2  | 1\.24%  |
| Missing                              | 66 | 40\.99% |
| Unparseable because of invalid UTF-8 | 3  | 1\.86%  |

> Only about one in five discovered files has a currently valid expiry within the RFC's recommended one-year horizon.

---

### A typo preserved by the National Museum

There is a nice typo on `nm.cz`:

```
Canonical: https://www.nm.cz/.well-known/sercurity.txt
```

I wouldn't expect to see **sercurity** in the **security** file, but here we are. The Czech National Museum has preserved a typo for future generations :)

Also nobody agrees how to spell Acknowledgments.

The registered field uses American spelling:

```
Acknowledgments:
```

`WEDOS.cz` and `CSFD.cz` use British spelling:

```
Acknowledgements:
```

`Rohlik.cz` uses singular:

```
Acknowledgement: /.well-known/acknowledgement.txt
```

These are syntactically permitted extension fields, so they do not invalidate a file by themselves, but software looking for the registered field ignores them.

`O2.cz` takes it one step further:

```
Acknowledgements:
Policy:
Signature:
```

All three fields are empty.


### Languages are difficult

Quick history lesson for foreign readers. We used to be Czechoslovakia. We are now the Czech Republic. But nowadays we prefer the short form, Czechia. So Czechia is a country; Czech is a language.

- Six domains confused `cz`, the country code for Czechia, with `cs`, the language code for Czech.
- Decathlon used human-readable language names instead of RFC 5646 tags.
- Komerční banka used a semicolon where RFC 9116 requires comma-separated tags.

```
allwyn.cz          Preferred-Languages: en, cz
ceskatelevize.cz   Preferred-Languages: cz, sk, en
sazka.cz           Preferred-Languages: en, cz
martinus.cz        Preferred-Languages: sk, cz, en
synottip.cz        Preferred-Languages: en, cz, sk
moneta.cz          Preferred-Languages: en, cz, sk
decathlon.cz       Preferred-Languages: English, French
kb.cz              Preferred-Languages: en; cs
```

It is especially amusing that the affected group includes two banks, a public service television broadcaster, and two national lottery/gambling brands. Meanwhile, the most common valid declaration was `cs, en`, used by 37 domains.

> It's funny that more sites specify which language to use than when the file becomes stale. More sites advertise jobs than publish a disclosure policy.

### UTF-8 encoding is even more difficult

Speaking of lottery/national gambling brands.

Here is a particularly good one: `tipsport.cz`, `chance.cz`, and `maxa.cz` all return the same body:

```
"Contact: mailto:appsec@tipsport.cz"
```

But they serve it as:

```
Content-Type: text/html
```

---

One invisible byte breaks three files:

`zive.cz`, `autorevue.cz`, and `dama.cz` serve byte-for-byte identical files. They look normal when decoded as Windows-1250:

```
# Czech News Center - reporting security vulnerabilities to the company
 
Contact: mailto:info@cncenter.cz
Contact: https://www.cncenter.cz/kontakt
OpenBugBounty: https://openbugbounty.org/bugbounty/marekl/
 
Hiring: https://www.cncenter.cz/kariera
Preferred-Languages: en, cs, sk
 
Expires: 2027-01-31T22:59:00.000Z
```

But their visually blank lines contain a raw `0xA0` non-breaking space. In UTF-8 that character must be encoded as `C2 A0`. Consequently, all three files fail UTF-8 decoding because of an invisible character.

---

ABOUT YOU drew a masterpiece, then forgot one newline.

`aboutyou.cz/.well-known/security.txt` has the largest found file at 2,648 bytes. Most of it is elaborate Unicode art inside comments, followed by a perfectly reasonable file:

```
Contact: mailto:security@aboutyou.com
Expires: 2027-10-01T00:00:00.000Z
Preferred-Languages: en, de
Hiring: https://corporate.aboutyou.de/en/departments/tech#jobs
```

The Unicode art is valid. Its only error is that the file does not end with a newline.

> 2\.6 KiB of Unicode artwork, defeated by one missing `\n`.

It also says automated scanning and fuzzing are not allowed. Nicely demonstrating that security.txt does not itself grant testing permission.

### Location and Redirects to nowhere

The location of security.txt is fairly simple. It must be under "/.well-known/" path and must use the "https" scheme.

> Retrieval of "security.txt" files and resources indicated within such files may result in a redirect (as per Section 6.4 of [RFC7231]). Researchers should perform additional analysis (as per Section 5.2) to make sure these redirects are not malicious or pointing to resources controlled by an attacker.

How hard is that? :)

Let's take a look at the Ministry of Finance's recursive localization:

```
curl -I https://mfcr.cz/.well-known/security.txt

HTTP/2 308
location: https://mf.gov.cz/.well-known/security.txt

---

curl -I https://mf.gov.cz/.well-known/security.txt

HTTP/2 302 
location: 

---

curl -I https://mf.gov.cz/.well-known/cs/.well-known/security.txt

HTTP/2 302 
location: cs/.well-known/cs/.well-known/security.txt

...
```

The path effectively grows recursively until it breaks.

---

Master Internet does every redirect mistake at once.

`master.cz` cycles through protocol, hostname and trailing-slash rewrites:

```
HTTPS without slash
→ HTTP with slash
→ HTTPS with slash
→ HTTPS on www with slash
→ HTTPS on www without slash
→ HTTP on www with slash
→ ...
```

Each rewrite rule fixes one property while breaking another, creating a loop.

---

But my favorite is when a missing slash becomes a new domain. When I started this, I wasn't expecting to find an open redirect.

```
curl -I https://lindabstrechy.cz/.well-known/security.txt

HTTP/2 301 
location: https://rovastrechy.cz.well-known/security.txt
```

It might be worth it to scan for an off-by-slash errors next ;)

---

One of our main pharmacies is also interesting.

`lekarna.cz` follows this redirect chain:

```
https://lekarna.cz/.well-known/security.txt
→ http://www.lekarna.cz/.well-known/security.txt
→ https://www.lekarna.cz/.well-known/security.txt
```

Its security contact file takes a brief trip through plaintext before returning to HTTPS.

While `modio.cz` just redirects from HTTPS to HTTP and stays there.

### Canonical commonly points somewhere else

There are 26 files with `Canonical` mismatch warning.

Examples include:

- Seven Shoptet customer domains pointing to Shoptet's own security.txt.
- Provider-managed sites pointing to SolidPixels, NUX or other suppliers.
- Czech domains pointing to parent `.com`, `.de` or `.sk` sites.
- `eshop-rychle.cz` pointing to the legacy `/security.txt` path.
- `nm.cz` pointing to `sercurity.txt`.
- `phoca.cz` pointing to GitHub.

The [RFC's Canonical semantics](<https://www.rfc-editor.org/rfc/rfc9116.html>) are unusually consequential: if the retrieved URI is not among the Canonical values, the file should not be trusted. Many people appear to treat it as “the main company security.txt” instead.

> If this field appears within a "security.txt" file and the URI used to retrieve that file is not listed within any canonical fields, then the contents of the file SHOULD NOT be trusted.

So yeah, all these companies went out of their way to add a field, which isn't mandatory, and by doing so, made their security.txt file not to be trusted for reporting.

```
curl https://embedit.cz/.well-known/security.txt
curl https://homecredit.cz/.well-known/security.txt
```

And it's really weird that some of the Czech companies with the largest security teams and 24/7 SOC delegate their security.txt to web and marketing agencies. I guess they are still responsible for the frontend vulnerabilities, but might receive something juicy :)

### Cryptography does not fix the basics

Only ten inline PGP-signed files were found. None passed strict validation.

- `cuni.cz` is otherwise excellent, signed file fails only because of the charset header.
- `annonce.cz` reaches its signed file through an HTTPS → HTTP → HTTPS redirect chain.
- `PostSignum.cz` publishes a valid-looking fingerprint without the required `openpgp4fpr:` URI scheme:

Their `Encryption` value is:

```
0127D7FB4BDF485424646A7317C45B9D22472FB3
```

That appears to be a 40-character OpenPGP fingerprint, but RFC 9116 requires `Encryption` to contain a **URI pointing to the key**, not a bare fingerprint. The RFC explicitly provides this format:

```
Encryption: openpgp4fpr:0127D7FB4BDF485424646A7317C45B9D22472FB3
```

Alternatively, they could use an HTTPS URL:

```
Encryption: https://www.postsignum.cz/path/to/public-key.asc
```

So they are only missing the `openpgp4fpr:` URI scheme. The fingerprint itself may be perfectly legitimate, but without the scheme it is not an absolute URI and therefore not valid RFC 9116 syntax. [RFC 9116 Encryption field](<https://www.rfc-editor.org/rfc/rfc9116.html#section-2.5.4>)

Especially funny for a national certificate authority. Their cryptographic identifier is probably fine, but the Encryption field is not.

### Contact is supposed to be machine-readable

We are near the end of the research. Contact is the whole point!

If you don't have a security.txt file, go and add one now. If you already have one, make sure it contains a valid way to contact you!!

**33 of the 161 discovered files (20.50%)** failed at least one Contact-related validation check.

- 4 had no parsed `Contact`
- 23 used bare email addresses without `mailto:`
- 7 contained another invalid Contact URI, such as whitespace or prose

For example Starbucks has a policy but no contact.

`starbucks.cz` provides:

```
Policy: https://hackerone.com/starbucks
Expires: 2026-12-31T00:00:00.000Z
Preferred-Languages: en
```

Which makes sense, but is not valid.

---

`AirBank.cz` put a space into every email URI:

```
Contact: mailto: web@airbank.cz
Contact: mailto: bugreport@airbank.cz
Contact: mailto: security@siteone.cz
```

---

`Decathlon.cz` wrote instructions instead of URIs

```
Policy: link vdp.decathlon.net
Contact: form link https://vdp.decathlon.net/p/Send-a-report
Preferred-Languages: English, French
Canonical: .well-known/security.txt
```

---

There was one, genuinely good unusual example. Email, telephone, X direct message and Facebook Messenger. And the complete file passes strict validation. Only one positive example between the broken ones.

---
<br>

## Conclusion

I started this because I discovered that my own `security.txt` files were broken. After scanning 1,000 popular `.cz` domains, I found 161 files, of which 20 passed strict RFC 9116 validation.

Apparently, I'm not alone :)

A failed check doesn't mean an organization has a bad security team, and a valid file doesn't guarantee that anyone will answer.

If you maintain a website, take a minute to fetch your `/.well-known/security.txt` file. Check the response headers, **the contact**, and the expiry date. The tool I shared might help you do just that.

DISCLAIMER: No websites were harmed during the making of this blog post. No companies were notified either. Feel free to reach out to them and collect some $$ bounties. Or don't.
