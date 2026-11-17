# SQL Injection - My Lab Writeup

By: eufr4nc-SEC

## What I was trying to do
I kept hearing "SQL Injection" everywhere in cybersecurity, so I wanted to actually see it with my own eyes. Not just read about it, but understand how a few characters typed into a login box can break a whole website.

## How I did it
I went to the login page of the lab.

At first I tried a normal username and password, just to see how it behaves when it fails. It said "invalid login" like normal.

Then I tried the classic test:
' OR '1'='1

I typed that into the username field, put anything as the password, and clicked login.

And it let me in. No real password. No real account.

That was crazy to me. I just sat there for a second like... "wait, that actually worked?"

Later I tried another one to see what else I could pull out:
' UNION SELECT username, password FROM users --

That one showed me how an attacker can start pulling data out of the database, not just bypassing login.

## Why it worked
The website was taking what I typed and putting it straight into its database query without checking it.

So when I typed `' OR '1'='1`, the database didn't see it as a username anymore. It saw it as part of its own instruction. And since `'1'='1` is always true, it thought I was a valid user.

It's like the site was letting me finish its sentence for it.

## How to fix it
If I was building this site, I would:
- Use prepared statements / parameterized queries, so user input is never treated as code
- Never build SQL queries by joining strings together
- Give the database user the least privilege possible, so even if someone gets in, they can't do much
- Validate and limit what users can type, but not rely on that alone

## What I actually learned
This lab changed how I look at login pages.

Before, I thought hacking was about complicated tools. This was just typing. That's what made it scary.

It also taught me that the most dangerous bugs are not always fancy. Sometimes they are just a developer trusting user input too much.

I want to keep learning this side of security - how to break it, and more importantly, how to fix it.
