# cfresearch
A repository containing my research from CloudFlare's AntiDDoS, JSChallenge, Challenges, and WAF.

## Challenge (Captcha)
- This is used for UAM (Under Attack Mode) and Challenge
- Cloudflare uses a script to provide access to the website while this rule is on. This is also how UAM works for CloudFlare. 
`cdn-cgi/l/chk_captcha` Is the container for the script cloudflare uses. The script, written in PHP, gives the visitor temporary access to the website with cftokens and cookies. `/cdn-cgi/l/chk_captcha?s=&g-recaptcha-response=&cv_chal_result=%&cv_chal_fp=&bf_challenge_id=&bf_execution_time=&bf_result_hash=`is the full URI for the request to the captcha. By generating random data into the fields, you could bypass the Captcha page for usage in DDoS attacks, or for web crawling.
> To Patch: `Follow Steps at Bottom`

## JS Challenge
- This is the 5 second wait before loading webpage
- Cloudflare also implements a script to 'check' the 'browser' as an attempt to stop ddos attacks. `dn-cgi/l/chk_jschl` Is the container for the script cloudflare uses. The script, written in php, gives the visitor full access to the website if a JS Challenge is implemented using cookies and tokens. `cdn-cgi/l/chk_jschl?s=&jschl_vc=&pass=jschl_answer=`
`s=cloudflarecookie, jschl_vc=cookie, pass=randomint with last 5 rayid, jschl_answer=randomint`
> To Patch: `Follow Steps at Bottom`

## Cache Bypass
- Cloudflare uses Edge servers to store cache and send requests to the webserver. If the server is caching pages such as HTML or PHP, or any content available via URI, you could send a `must-revalidate`query in your header to make the Cloudflare Edge server revalidate the cache and GET a new file regardless if the file has changed or not.
> To Patch: `Remove must-revalidate or max-age from Cache-Control header`

## Authentic Traffic
- To get allowed access to cloudflare regularly, you need a CloudFlare User ID cookie, labeled `__cfduid`. These can be obtained by setting the bot to allow cookies and request a cookie. Some application will require a `PHPSESSID` cookie to obtain access also.
> Patch: Limit Traffic To Webserver | RateLimiting | Limit Connections | Kill Connections

## Patching Layer7 Attacks
- Patching Layer7 Attacks can be hard, but with proper setup, can be very easy. First, start by ensuring visitor IP's. This can be done in both Apache and Nginx by using a cloudflare module to log the visitor ip, instead of cloudflare's IP. Second, setup ratelimiting for the website and install fail2ban to help enforce and jail the ratelimiting offenders. Third, block ASNs  that are known for DDoS Attacks, such as ChinaNet, and various Backbones of eastern countries such as China, Japan, and Russia. (Note: Blocking ASNs can Block some real user traffic) A very easy patch can be to cache the pages and force cloudflare to serve these files, rather than your webserver serving the files. 

- If you are getting targetted by a person individually, they are probably stupid. Look for patterns in their attack, such as proxies, spoofed IPs, UserAgents, IP Ranges, IP ASNs, etc. Usually, attackers do not randomize this data and leave this data available in the headers. Setup a firewall rule to block this common feature of the attacks, such as a common user agent, or common proxy server. This can take a huge load off your webserver and put it on cloudflare. Cloudflare's network is built for that kind of stress, so let cloudflare do its job.



