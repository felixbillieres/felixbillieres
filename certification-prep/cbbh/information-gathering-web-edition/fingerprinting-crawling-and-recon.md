# Fingerprinting, Crawling & recon

Fingerprinting focuses on extracting technical details about the technologies powering a website or web application.

Fingerprinting serves as a cornerstone of web reconnaissance for several reasons:

* `Targeted Attacks`: By knowing the specific technologies in use, attackers can focus their efforts on exploits and vulnerabilities that are known to affect those systems. This significantly increases the chances of a successful compromise.
* `Identifying Misconfigurations`: Fingerprinting can expose misconfigured or outdated software, default settings, or other weaknesses that might not be apparent through other reconnaissance methods.
* `Prioritising Targets`: When faced with multiple potential targets, fingerprinting helps prioritise efforts by identifying systems more likely to be vulnerable or hold valuable information.
* `Building a Comprehensive Profile`: Combining fingerprint data with other reconnaissance findings creates a holistic view of the target's infrastructure, aiding in understanding its overall security posture and potential attack vectors.

For fingerprinting you can use techiniques such as ->

* Banner grabbing
* Analysing HTTP Headers
* `Probing for Specific Responses`
* `Analysing Page Content`

With Tools such as Wappalyser, nmap ou others

#### Banner grabbing command ->

```
ElFelixi0@htb[/htb]$ curl -I inlanefreight.com

HTTP/1.1 301 Moved Permanently
Date: Fri, 31 May 2024 12:07:44 GMT
Server: Apache/2.4.41 (Ubuntu)
Location: https://inlanefreight.com/
Content-Type: text/html; charset=iso-8859-1
```

we can see that it tries to redirect us to https://inlanefreight.com

```
ElFelixi0@htb[/htb]$ curl -I https://www.inlanefreight.com

HTTP/1.1 200 OK
Date: Fri, 31 May 2024 12:12:26 GMT
Server: Apache/2.4.41 (Ubuntu)
Link: <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Link: <https://www.inlanefreight.com/index.php/wp-json/wp/v2/pages/7>; rel="alternate"; type="application/json"
Link: <https://www.inlanefreight.com/>; rel=shortlink
Content-Type: text/html; charset=UTF-8
```

so here we get some other informations, we know it's a wordpress that is doing the redirection

## Crawling

`Crawling`, often called `spidering`, is the `automated process of systematically browsing the World Wide Web`. Similar to how a spider navigates its web, a web crawler follows links from one page to another, collecting information.

Crawlers can extract:

* `Links`
* `Comments`
* `Metadata`
* `Sensitive Files`

## robots.txt

Robots.txt acts as a virtual "`etiquette guide`" for bots, outlining which areas of a website they are allowed to access and which are off-limits.

Let's imagine the following:

```txt
User-agent: *
Disallow: /private/
```

This directive tells all user-agents (`*` is a wildcard) that they are not allowed to access any URLs that start with `/private/`.

`User-agent`: This line specifies which crawler or bot the following rules apply to. A wildcard (`*`) indicates that the rules apply to all bots. Specific user agents can also be targeted, such as "Googlebot" (Google's crawler) or "Bingbot" (Microsoft's crawler).

Let's imagine the following:

```txt
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/

User-agent: Googlebot
Crawl-delay: 10

Sitemap: https://www.example.com/sitemap.xml
```

* All user agents are disallowed from accessing the `/admin/` and `/private/` directories.
* All user agents are allowed to access the `/public/` directory.
* The `Googlebot` (Google's web crawler) is specifically instructed to wait 10 seconds between requests.
* The sitemap, located at `https://www.example.com/sitemap.xml`, is provided for easier crawling and indexing.
