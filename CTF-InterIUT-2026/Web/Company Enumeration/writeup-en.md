# Company Enumeration - Web

(Auto translated using mistral, do tell me if anything isn't clear and / or straight up wrong)

## Description

An internal portal of MilSec Industries allows employees to consult members of partner companies that the corporation collaborates with.

However, it is impossible to log in to the portal.

After analyzing the traffic, Null-Syndicate discovered that an internal API is used to retrieve this information.

The portal is blocked, but the API might not be...

----

This writeup follows my *exact* thought process. If an idea seems stupid or completely unrelated to the solution, don’t worry—that’s just me being the idiot here.

The first thing I noticed was the login form token. Just in case, I saved it. I thought I might need to add a header like `Authorization: Bearer <id>`.

```html
<input type="hidden" id="login_form__token" name="login_form[_token]" value="03136567a260e173b5667ff3.d2MhBmkiAyfcA9Jh3qwcyUdFxRyRcHqoxY6zg0KrbIg.QVUTQlhnTxOFVKsgtPhVv34RhES8QwDN_Obewir7CsAGBmhUGloyQ5FpuQ">
```

To check for endpoints, I used good old `gobuster` with a small wordlist—otherwise, the firewall sends me back to the Stone Age:

```bash
┌──(venv)─(kali㉿kali)-[~/Flask]
└─$ gobuster dir -u "http://node2.interiut.hack2g2.fr:12792" -w /usr/share/wordlists/dirb/common.txt 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://node2.interiut.hack2g2.fr:12792
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htaccess            (Status: 403) [Size: 333]
.htpasswd            (Status: 403) [Size: 333]
.hta                 (Status: 403) [Size: 333]
assets               (Status: 301) [Size: 388] [--> http://node2.interiut.hack2g2.fr:12792/assets/]
auth                 (Status: 200) [Size: 6517]
index.php            (Status: 301) [Size: 381] [--> http://node2.interiut.hack2g2.fr:12792/]
server-status        (Status: 403) [Size: 333]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
```

Code found in the auth:
![](./members-enpoint.png)

From there, we can see there’s an API to test at `/members`.
Given the challenge name is **"Enumeration"**, it’s likely we just need to brute-force test all possible members like idiots.

Testing with an empty `GET` request returns:
```json
{
    "success": false,
    "msg": "Not Authorized"
}
```

I tested the following endpoints:
- `/members/id_ici`
- `/members/list`
- `/members/id/id_ici`
- `/members/cestquoicetteapiencoresuccessful/id_ici`
- `/members/envidecanner`

And then came a *crucial* moment: **thinking for two seconds**.
Because, you see, my pebble-sized IQ had slightly forgotten that APIs are typically tested with `POST`, not `GET`. No big deal, it only took me 5 minutes to realize—but we won’t talk about that.

At this point, all I had to do was test requests with BurpSuite Intruder and... stare at the screen waiting for the flag to pop up.

So, the main vulnerability here is an **IDOR**—this classic API vulnerability (**shoutout to ANTS**)—which allows access to objects without authentication. There you go, kids. Always remember to restrict API access with auth.

Eventually, we find the flag in plaintext.

**Note:** A Python script that checks for the string `"inter"` or similar would have been *way* more efficient. Don’t be like me—be smart.

![](./flag.png)

## Flag: `interiut{id0r_Unpr0tected}`