# Vault - 4-Digit PIN Brute Force (No Rate Limit) + Base64 Decode

By: eufr4nc-SEC
Category: Web Security / Authentication / Brute Force
Tool Used: Burp Suite lab mode/ Community - Proxy, Intercept, Intruder, Decoder
Difficulty: Easy to Medium
Status: Vulnerable - Successfully Exploited

## 1. Objective

The objective of this lab was to test the security of a web-based vault that was protected by a 4-digit PIN.

I wanted to answer three questions:

1. Does the application limit how many times I can guess the PIN?
2. Can I automate the guessing using Burp Suite Intruder?
3. Once I get inside, is the secret actually protected, or is it just encoded?

This lab is important because many real applications still use short PINs for sensitive actions - bank apps, vaults, admin panels, password reset codes, OTP codes. If they do not have rate limiting, they are all vulnerable in the same way.

## 2. Initial Reconnaissance

When I first opened the vault page, I saw a simple input box asking for a PIN and an Unlock button. There was no information about how many attempts I had, no CAPTCHA, and no warning about lockout.

I tested a few manual PINs to understand the behaviour:

- Tried `1234` - Failed. Error: "Incorrect PIN"
- Tried `0000` - Failed. Error: "Incorrect PIN"
- Tried `1111` - Failed. Error: "Incorrect PIN"

What I noticed was very important:

1. The error message was always the same. It did not tell me if I was close.
2. The page loaded just as fast every time. There was no delay.
3. After 3, 5, 10 failed attempts, nothing happened. No lockout, no CAPTCHA, no "Too many attempts" message.
4. I could keep trying forever.

At this point I already had a strong suspicion: This application has no rate limiting.

A 4-digit PIN only has 10,000 possible combinations, from 0000 to 9999. That is a very small number for a computer. A human cannot try 10,000 PINs by hand, but a tool can do it in minutes if the application lets it.

## 3. Intercepting the Request with Burp Suite

To automate the attack, I needed to understand how the application sends the PIN to the server.

**Steps I followed:**

1. I opened Burp Suite and made sure the Proxy was running.
2. I configured my browser to use Burp as the proxy.
3. In Burp, I went to Proxy > Intercept and turned Intercept ON.
4. In the browser, I entered a test PIN `1234` and clicked Unlock.
5. Burp captured the request before it reached the server.

The captured request looked like this:
POST /vault/unlock HTTP/1.1
Host: http://target-lab.local
Content-Type: application/x-www-form-urlencoded

pin=1234
This confirmed that the PIN is sent as a simple POST parameter called `pin`. There was no CSRF token, no extra header, no encryption on the PIN itself. It was just plain text in the request body.

I forwarded the request and then right-clicked and selected **Send to Intruder**.

4. Configuring Burp Intruder with § Payload Markers §

Intruder is the tool in Burp Suite used for automated attacks. It takes a request and replaces a part of it with many different payloads, one after another.

In Intruder, Burp automatically highlights some values as payload positions. I cleared all of them first, because I only wanted to attack one place: the PIN.

I selected only the PIN value and clicked **Add §**.

The request now looked like this:

POST /vault/unlock HTTP/1.1
Host: http://target-lab.local
Content-Type: application/x-www-form-urlencoded

pin=§1234§

**What the § means:**

The `§` symbol is how Burp marks the payload position. Everything inside the `§ §` will be replaced during the attack. Everything outside it will stay exactly the same.

So Burp understood: "Keep the URL, keep the headers, keep `pin=` - but change only the number inside the § § for every request."

This is very important to understand, because if you leave extra § markers, Burp will try to attack the wrong places and the attack will fail.

5. Setting the Payloads: 0000 to 9999

In the Payloads tab, I configured:

- Payload type: Numbers
- From: 0000
- To: 9999
- Step: 1

This tells Burp to try every single number from 0000 to 9999, in order.

I also made sure the Payload Encoding was correct so that Burp would send `0000` and not just `0`. Leading zeros are important. If Burp sends `1` instead of `0001`, the application might reject it even if `0001` is the correct PIN.

I then clicked **Start Attack**.

6. The Attack - No Rate Limit, No Lockout

This is where the vulnerability became clear.

Burp started sending requests very fast. 100, 500, 1000, 5000 requests - and the server kept answering normally.

There was:
- No rate limiting - It did not slow me down
- No IP blocking - My IP was not blocked
- No CAPTCHA - It never asked me to prove I was human
- No account lockout - The vault never locked
- No
- ## . Prevention

- Limit to 5 failed attempts, then lock for 15 minutes
- Add CAPTCHA after 3 failed attempts
- Block IPs that send too many requests too fast
- Use longer PIN / password + MFA, not just 4 digits
- Store secrets with real encryption (AES-256), not Base64
- Log all failed attempts and alert the user

