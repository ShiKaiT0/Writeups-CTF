# Never Fade Away - Web

## Description

To prove your worth to **GhostSurface**—Null-Syndicate’s elite web hacking division—you must infiltrate one of HelixGate Systems’ public services. This portal is used by partner companies to manage their component orders. A Null-Syndicate member discovered that the application has a **major design flaw**. If you succeed, GhostSurface will be able to install a backdoor at HelixGate—this would be the first breach in the MilSec network...

----

The interface is either a very classic or *vibe-coded* login form—I don’t know, but there’s a big login button, so I tested a few SQL injections. They led nowhere.

So I created an account with *exceptional* security:
- **Account ID:** `test`
- **Email:** `test@test.test`
- **Password:** `**test**`

Looking at the JWT, I saw `{'user_id':'1'}`.

Example "JWT": eyJ1c2VyX2lkIjoxfQ.ahqdKg.2qC8Dph2zLiwoKH06NMv2-yVK5c

I have no idea how this is structured, but when I try to decode it, I get some *pretty absurd* results:

eyJ1c2VyX2lkIjoxfQ.ahqfWw.P36fO5IcKq8tOXTeOdmZoXCjAmQ =
{"user_id":1}\u0006\u00A1\u00A9\u00F5\u00B0?~\u009F;\u0092\u001C*\u00AF-9t\u00DE9\u00D9\u0099\u00A1p\u00A3\u0002d

Maybe some PHP file path trick? Or command chaining? `(?~\ .. ;)`
**Problem:** The string is barely readable as basic ASCII.

Let’s try another one to see the differences:

eyJ1c2VyX2lkIjoxfQ.ahqjIg.0PfNluZHz9ZFMZHBqUc6egQ1j6g
{"user_id":1}\u0006\u00A1\u00AA2 \u00D0\u00F7\u00CD\u0096\u00E6G\u00CF\u00D6E1\u0091\u00C1\u00A9G:z\u00045\u008F\u00A8


So probably a dead end? ***But what the hell is this?***

Well, this doesn’t look like a standard JWT. Maybe it’s a format I don’t know, but even [jwt.io](https://jwt.io) is completely lost—so it’s probably some custom thing. Not really worth investigating.

I launched a brute-force attempt in the background, without much hope. And sure enough, it didn’t work.
```bash
┌──(venv)─(kali㉿kali)-[~/Flask]
└─$ python flask_unsign --unsign --cookie "eyJ1c2VyX2lkIjoxfQ.ahqo8g.KGyA4CSDPbpb8it2KcDeZYoVL7g" -w /usr/share/wordlists/rockyou.txt --no-literal-eval
[*] Session decodes to: {'user_id': 1}
[*] Starting brute-forcer with 8 threads..
[!] Failed to find secret key after 14344392 attempts.
```

A quick scan reveals an /admin endpoint.
![](./gobuster.png)

There’s something that checks for is_admin=true. I tried several headers and cookies: is_admin, admin, ADMIN, IS_ADMIN.

![](./admin_panel.png)

I started testing with this command:

```bash
curl -v -b "session=eyJ1c2VyX2lkIjoxfQ.ahrAXQ.cxWNnRwd3L8dnHVwG9k8B9rSfJ8;is_admin=true;admin=true" -H "is_admin: true; admin: true" -X GET "http://node3.interiut.hack2g2.fr:13660/admin"
```

### Vulnerability type (hint): Mass Assignment

## SOLUTION FOUND:
Actually, considering the Mass Assignment hint and reading the docs, I looked for the only place where user data was set—which turned out to be /register.
In Burp Suite, you can see the fields, and all we need to do is add our own field.

![](./burp-mass-asignment.png)

### Result:
![](./flag.png)

## Flag: interiut{m45s_455ignm3nt_pwn3d_g4st5urf4c3}
Useful Resources:
[PayloadsAllTheThings - Mass Assignment](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Mass%20Assignment)