Cap
### IDOR

An [Insecure Direct Object Reference](https://portswigger.net/web-security/access-control/idor) (IDOR) is a vulnerability there an attacker can manipulate a url or parameter to a request to access objects that they were not intended to access. These bugs seem trivial, but are all over the place (like [US Department of Defense](https://www.zdnet.com/article/bug-hunter-wins-researcher-of-the-month-award-for-dod-account-takeover-bug/), [political party websites](https://grahamcluley.com/alex-salmonds-alba-party-website-leaks-data-in-idor-foul-up/), [ZenDesk](https://www.bleepingcomputer.com/news/security/typeform-fixes-zendesk-sell-form-data-hijacking-vulnerability/), and [Parler](https://www.wired.com/story/parler-hack-data-public-posts-images-video/)).

In this case, I have a link to `/download/7`. But if I start to step back, I can find other PCAPs.

I’ll exploit this with a quick loop to get everything. If I notice that that number seems to one up, I’ll download until it fails, and then break, with the following loop:

```
for i in {0..500}; do 
  wget 10.10.10.245/download/${i} -O pcaps/${i}.pcap 2>/dev/null || break; 
done;
```