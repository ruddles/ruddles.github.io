---
title: Hacker 101 CTF Micro-CMS v2 - Flag 1
published: true
---

Below are my notes on how I found the first flag on the Hacker 101 CTF Micro-CMS v2 (listed as moderate difficulty). If you're looking to follow along then I assume you're all logged in and have hit Go on the challenge. It's been a while since I've done one of these so there's some dead ends along the way.

# Finding a vulnerability

So clicking around I can see that they've added authentication to the site now (probably a good move). Attempting to edit or create a page take you to the login:

![login page]({{ site.url }}/assets/post-images/h101-cms-v2/login.png)

Interesting, my first thought is sql injection, so lets enter `'` as the username and see if we can break it.....

![login error]({{ site.url }}/assets/post-images/h101-cms-v2/LoginError.png)

OK so this is blatting the error message out to the user, nice. Looks like there's a sql injection vuln here to play with.

At this point I went down a bit of a rabbit hole looking for usernames, including starting to run a brute force attack on usernames with the list from https://github.com/jeanphorn/wordlist. I'll spare you the time, that's not the way to go here, honestly I don't even know what I'd do with the username once I found it.

# SQL Injection

OK so going back to the SQL, it looks like I need to be able to control the password value returned by the SQL statement. After digging through some SQL injection cheat sheets it hit me, I'm an idiot, all I need is a UNION statement. If you've not come across one of these before it basically allows you to combine 2 result sets - the rule being they need to have the same columns in both sets. In this case there's only 1 column in the result set: password.

So with that in mind I need to create a union between an empty result set (either by entering a username that doesn't exist - I have plenty of those - or in this case an empty username works just as well) and a single password I control: `UNION SELECT '123' as password`. Now just to add the ' at the front to escape out of the username entry, and # at the end to comment out the rest of the script and we end up with:

Username: `' UNION SELECT '123' as password #`

Password: `123`

# Logging in

OK so entering those login details we get in!

![we're in]({{ site.url }}/assets/post-images/h101-cms-v2/LoggedIn.png)

Looks like there's a private page, and yup it has a flag (which I won't post here).

Any comments or suggestions? Chat to me on [twitter](https://twitter.com/RuddlesDev)
