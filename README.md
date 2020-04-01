# cfresearch
A repository containing my research from CloudFlare's AntiDDoS, JSChallenge, Challenges, and WAF.
This was built for educational purposes such as learning how CloudFlare works, how to bypass CloudFlare, and how to prevent L7 DDoS attacks.
> Contact Me: https://t.me/trespassed

> Other Research: https://github.com/scaredos/l7research (Not Finished, Finishing CloudFlare research first)

## Update (CloudFlare uses hCaptcha)
CloudFlare now uses hCaptcha instead of reCaptcha. The URI has not changed, but the post data has. This disables support for Privacy Pass since it is not the Google Captcha. hCaptcha is a replacement for other Captcha challenges which advertises itself as a way for website owners to earn money from blocking bots. hCaptcha uses the users input to train machine learning models and neural networks, making the businesses that own the machine models pay.

`r`: Logging information (Not Required)

`cf_captcha_kind`: 'h' or 're' (hCaptcha or reCaptcha)

`id`: ID of the request

`g-recaptcha-response`: Response of the hCaptcha

`h-recaptcha-response`: Response of the hCaptcha

## CloudFlare DDoS Introduction
The common Layer7 DDoS attack are quite easy to pull off and most of the time work perfectly. In this write-up, I will explain features of CloudFlare and methods attackers use to (D)DoS your website. If your website is being (D)DoSed and you use CloudFlare, you have incorrectly configured CloudFlare. CloudFlare's network is not perfect and has some issues, but is excellent for handling (D)DoS attacks like this. By default, CloudFlare protects you from Layer 4 attacks by proxying all of your traffic through their servers. Down below I mention some ways to stop your website from being attacked.
- > NOTE: JS Bypasses are the most common since they are the easiest to create and use.

## Challenge (Captcha) (NEW)
- The new CloudFlare captcha was introduced recently in part of an attempt to block Layer7 DoS attacks
- The new method is `POST` to `?__cf_chl_captcha_tk__=GENERATED_TOKEN`. It hands a `cf_clearance` cookie, allowing the user to bypass captcha, to the accepted device and, as usual, a `__cfuid` cookie stating the CloudFlare visitor id. The `cf_clearance` expires 1 day after the cookie was given and is valid for over 1k requests or until CloudFlare forces you to captcha again.
- The form data for the POST request is 'r', which was discovered to be used for analytics by CloudFlare, 'cf_captcha_kind', which indicates which captcha the user solved, 'id', the visitor id, 'g-recaptcha-response', which is the hCaptcha response or reCaptcha response, and 'h-recaptcha-response', which is the hCaptcha response or reCaptcha response.

> To Patch: `Follow Steps at Bottom`

## Challenge (Captcha) (Outdated Information)
- This is used for UAM (Under Attack Mode) and Challenge
- Cloudflare uses a script to provide access to the website while this rule is on. This is also how UAM works for CloudFlare. 
`cdn-cgi/l/chk_captcha` Is the container for the script cloudflare uses. The script, written in PHP, gives the visitor temporary access to the website with cftokens and cookies. `/cdn-cgi/l/chk_captcha?s=&g-recaptcha-response=&cv_chal_result=%&cv_chal_fp=&bf_challenge_id=&bf_execution_time=&bf_result_hash=`is the full URI for the request to the captcha. By generating random data into the fields, you could bypass the Captcha page for usage in DDoS attacks, or for web crawling.
> To Patch: `Follow Steps at Bottom`

## JS Challenge (NEW)
- This is used for UAM (Under Attack Mode)
- CloudFlare implements a "Browser Check" to generate a token for cloudflare clearance. Upon solving the JS Challenge, you are given a `__cfuid` cookie for CloudFlare Visitor Logging and `cf_clearance` to bypass the JS Challenge upon loading. The new URI is `?__cf_chl_jschl_tk__=GENERATED_TOKEN` and no longer is stored in `/cdn-cgi/`. 
- `POST` request with `jschl_vc:` which is the JS Challenge ID, `pass:` which is the "pass" for CloudFlare's server, and `jschl_answer:` which is the solution to the given JavaScript Challenge.
CloudFlare has succesfully blocked the Python CloudFlare scraping module known as `cfscrape` from scraping pages behind UAM.
> To Patch: `Follow Steps at Bottom`

## JS Challenge (Outdated Information)
- This is the 5 second wait before loading webpage
- Cloudflare also implements a script to 'check' the 'browser' as an attempt to stop ddos attacks. `dn-cgi/l/chk_jschl` Is the container for the script cloudflare uses. The script gives the visitor full access to the website if a JS Challenge is implemented using cookies and tokens. `cdn-cgi/l/chk_jschl?s=&jschl_vc=&pass=jschl_answer=`
`r=cloudflarecookie, jschl_vc=cookie, pass=randomint with last 5 rayid, jschl_answer=randomint`
> To Patch: `Follow Steps at Bottom`

## Raw Power (r/s)
With enough devices across enough locations with different ISPs and User Agents, you could shut down a website by bypassing the cache servers. Most likely, the site you are hitting doesn't recieve global traffic from 1k devices in each third world country. CloudFlare's cache servers in those countries will not have the website stored, therefore bypassing the caching process of CloudFlare, leaving the target webserver vulnerable. Using raw power along with the "Cache Bypass" method is very effective.
> To Patch: `Test your server weekly with CFBypass DDoS attacks. It will help the CloudFlare servers cache your content in small countries`


## NGINX PHP Application w/ CloudFlare Bypass
- The easiest target when attacking a website is the PHP pages. If you're server runs NGINX and PHP, then you're most likely vulnerable to this. If you get a `502 Bad Gateway` when your website gets hit, it's just a server error, nothing is wrong with your server. Either CloudFlare, or (most likely) your server's PHP queue is full. A simple `service php7.0-fpm restart` will fix it temporarily, but for a permanant fix, cache your PHP pages, setup limit_conn, proxy_cache, rate_limit, and other tools in NGINX's configuration for your php pages. Caching your PHP pages, then assuring the `cf-cache-status` is a `HIT`, will help your website take the attack and stop the 502 Gateway Errors.
> To Patch: `Cache PHP Pages | RateLimit | Limit Connections`

## Cache Bypass
- Cloudflare uses Edge servers to store cache and send requests to the webserver. If the server is caching pages such as HTML or PHP, or any content available via URI, you could send a `must-revalidate` or `max-age=0` or `no-cache no-store` query in your header to make the Cloudflare Edge server revalidate or reinstate the cache and GET a new file regardless if the file has changed or not.
- Another Cache Bypass method is random URI. Generating a random set of strings or integers and placing in the URL will bypass cached content (because it has never been loaded before). This is very effective as well, and with the script I uploaded (bypass.py), you can easily test this against your website if you have access to a server that will not ban you for DoS or have python install on your machine.
> To Patch Pt.1: `Remove must-revalidate or max-age from Cache-Control header and make CloudFlare reinstate the cache`

> To Patch Pt.2: `You must limit connections and make cloudflare cache the 404 page`

## Patching Layer7 Attacks
- Patching Layer7 Attacks can be hard, but with proper setup, can be very easy. First, start by ensuring visitor IP's. This can be done in both Apache and Nginx by using a cloudflare module to log the visitor ip, instead of cloudflare's IP. Second, setup ratelimiting for the website and install fail2ban to help enforce and jail the ratelimiting offenders. Third, block ASNs  that are known for DDoS Attacks, such as ChinaNet, and various Backbones of eastern countries such as China, Japan, and Russia. (Note: Blocking ASNs can Block some real user traffic) A very easy patch can be to cache the pages and force cloudflare to serve these files, rather than your webserver serving the files. 

- Look for patterns in attacka, such as proxies, spoofed IPs, UserAgents, IP Ranges, IP ASNs, etc. Usually, attackers do not randomize this data creating an easier way to stop the attack. Setup firewall rules to block the common features of the attacks, such as a common user agents, or a proxy server. This can take a huge load off your webserver and put it on cloudflare. Cloudflare's network is built for large requests.


