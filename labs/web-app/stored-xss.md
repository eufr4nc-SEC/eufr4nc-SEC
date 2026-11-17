# Stored XSS - My Lab Writeup

By: eufr4nc-SEC

## What I was trying to do
I wanted to understand how an attacker can leave a trap on a website that hits anyone who visits later. Not just one person, but everybody.

That's what Stored XSS is.

## How I did it
I went to a comment box on the lab site. At first I just typed normal text to see how it works.

Then I tried something small:
<script>alert(1)</script>

I clicked submit, reloaded the page, and the alert popped up. That was the moment I knew it worked.

The site saved my script like it was a normal comment. It didn't clean it. So every time someone opened that page, the script would run for them too.

Later I tried a stronger one to see if I could grab a cookie:
<script>alert(document.cookie)</script>

## Why it worked
The website trusted whatever I typed. It saved it straight to the database and showed it back to users without checking if it was dangerous.

That's the mistake. It should have cleaned the input and encoded the output.

## How to fix it
If I was building this site, I would:
- Clean user input before saving it
- Encode output so <script> is shown as text, not run as code
- Add a Content Security Policy to block scripts like this
- Never trust input just because it came from a form

## What I actually learned
Reflected XSS felt like a trick on one person. Stored XSS felt different. It felt like leaving something behind for the next person.

That's why it's more dangerous. The attacker doesn't even need to be online when the victim gets hit.

This lab made me start thinking like a defender too, not just like an attacker. And that's what I want to keep doing.
