---
layout: page
title: "Bug bounty"
permalink: /bug-bounty/
---

## HackerOne

* [HackerOne profile](https://hackerone.com/vavkamil){:target="_blank"}

### Higlights

* Evernote / [Automattic](https://hackerone.com/reports/550937){:target="_blank"} / Starbucks / [Node.js](https://hackerone.com/reports/550937){:target="_blank"}

## Bugcrowd

* [Bugcrowd profile](https://bugcrowd.com/vavkamil){:target="_blank"}

### Higlights

* Smartsheet / Quora / Fitbit / Spotify / Bugcrowd / LastPass / SendSafely / Tesla

## Write-ups & disclosed reports

* [All-in-One WP Migration < 7.15 - Arbitrary Backup Download](https://wpvulndb.com/vulnerabilities/10151){:target="_blank"}
* [Disclosure of invoices and PII via android app API (IDOR)](#todo)
* [Node.js ~ CRLF Injection in legacy url API (url.parse().hostname)](https://hackerone.com/reports/771596){:target="_blank"}
* [Insufficient DKIM record with RSA 512-bit key used on WordPress.com](https://hackerone.com/reports/550937){:target="_blank"}

## Responsible disclosure

| Website           | Company                      | Vulnerability                         | Details                                            | Reward | References                                                               |
|-------------------|------------------------------|---------------------------------------|----------------------------------------------------|--------|--------------------------------------------------------------------------|
| Airbank.cz        | Siteone s.r.o.               | XSS                                   | Reflected Dom XSS via POST request                 | Yes    | [security.txt](https://www.airbank.cz/.well-known/security.txt){:target="_blank"}          |
| Subreg.cz         | Gransy s.r.o.                | XSS                                   | Multiple Reflected & Dom based XSS                 | Yes    |                                                                          |
| Airbank.cz        | Siteone s.r.o.               | Unrestricted file upload              | Insecure API end-point allowing to upload any file | Yes    | [security.txt](https://www.airbank.cz/.well-known/security.txt){:target="_blank"}          |
| Active24.cz       | ACTIVE 24, s.r.o.            | XSS                                   | Dom based XSS in domain search                     | No     | [security.txt](https://www.active24.cz/.well-known/security.txt){:target="_blank"}         |
| Prospanek.cz      | SmartBase                    | XSS                                   | Reflected XSS in search                            | No     |                                                                          |
| Damejidlo.cz      | damejidlo.cz s.r.o.          | XSS                                   | Reflected XSS in restaurants archive               | Yes    |                                                                          |
| Gopay.com         | GOPAY s.r.o.                 | XSS                                   | Stored XSS in WordPress (CVE-2019-11869)           | No     | [security.txt](https://www.gopay.com/.well-known/security.txt){:target="_blank"}           |
| Trezor.io         | SatoshiLabs s.r.o.           | Domain takeover                       | Disclosure of user IPs via debug MessageEvent      | Yes    | [Leaderboard](https://trezor.io/security/){:target="_blank"}                               |
| Rohlik.cz         | Velká Pecka s.r.o.           | Improper Access Control               | Access to Google calendar revealing sensitive info | No     | [security.txt](https://www.rohlik.cz/.well-known/security.txt){:target="_blank"}           |
| o2.cz             | O2 Czech Republic a.s.       | Multiple IDORs                        | Disclosure of invoices and PII via android app API | No     | [Write-up](#todo)                                                        |
| Idos.cz           | MAFRA, a. s.                 | Multiple XSS, Arbitrary File Download | Source-code disclosure                             | No     |                                                                          |
| Socialnisystem.cz | Česká pirátská strana        | Reflected XSS                         | Found via source code review                       | No     | [GitHub issue](https://github.com/pirati-web/socialnisystem.cz/issues/1){:target="_blank"} |
| Eshop.upc.cz      | UPC Česká republika, s.r.o.  | Reflected XSS                         | Unvalidated redirect                               | No     |                                                                          |
| Cd.cz             | Ceske drahy, a.s.            | Personal information leak             | Misconfigured API (PII in JavaScript, CORS)        | Yes    |                                                                          |
| CSFD.cz           | POMO Media Group s.r.o.      | XSS, IDOR, account takeover           | Multiple vulnerabilities reported via bug bounty   | Yes    | [Hall of Fame](https://www.csfd.cz/vyvojari/){:target="_blank"}                            |
| T-mobile.cz       | T-Mobile Czech Republic a.s. | CSRF, Multiple XSS, ...               | Multiple vulnerabilities reported via bug bounty   | Yes    | [Hall of Fame](https://www.t-mobile.cz/bug-bounty/zed-slavy){:target="_blank"}             |
| Zive.cz           | CZECH NEWS CENTER a. s.      | XSS                                   | Multiple XSS vulnerabilities                       | No     | [security.txt](https://www.zive.cz/.well-known/security.txt){:target="_blank"}             |
| Mall.cz           | Internet Mall, a.s.          | XSS                                   | XSS in Angular Template                            | Yes    | [Hacktrophy](https://hacktrophy.com/){:target="_blank"}                                    |
| Fler.cz           | Fler s.r.o.                  | CORS misconfiguration                 | Disclosure of PII via JavaScript exploit           | No     |                                                                          |
| Alza.cz           | Alza.cz, a.s.                | XSS                                   | XSS filter bypass                                  | No     | [security.txt](https://www.alza.cz/.well-known/security.txt){:target="_blank"}             |
| Heureka.cz        | Heureka Group a.s.           | XSS                                   | Multiple XSS vulnerabilities                       | No     | [security.txt](https://heureka.cz/.well-known/security.txt){:target="_blank"}              |
| Drmax.cz          | Dr. Max BDC, s.r.o.          | CORS misconfiguration                 | Disclosure of PII via JavaScript exploit           | No     |                                                                          |
| Email.cz          | Seznam.cz, a.s.              | Account takeover                      | XSS -> CSP bypass -> CSRF -> account takeover      | No     |                                                                          |
| Tele3.cz          | TELE3 s.r.o.                 | SQLi                                  | Blind based SQL Injection                          | Yes    |                                                                          |
| Daybyme.com       | DayByMe s.r.o.               | CSRF, IDOR                            | Stored XSS with significant impact                 | Yes    | [Hacktrophy](https://hacktrophy.com/){:target="_blank"}                                    |
| Tsbohemia.cz      | T.S.BOHEMIA a.s.             | XSS                                   | Dom based XSS in search function                   | No     |                                                                          |