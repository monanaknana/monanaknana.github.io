---
title: "Atan: Garage Leak CTF Writeup Guide"
date: 2026-05-19 10:30:00 +0800
categories: [CTF, Web]
tags: [lfi, file-inclusion, web-security, atur-kreatif]

image:
  path: /assets/img/posts/atan/banner.png
  alt: Atan Garage Leak LFI CTF Challenge

pin: true
toc: true
comments: true
---

# Atur Kreatif’26 Web - Atan: Garage Leak

---

This is the overview step by step to solve this question. This challenge involves Local File Inclusion (LFI) a vulnerability where a web application uses user controlled input to load files on the server without proper validation. When the `?page=` parameter accepts filenames directly, an attacker can manipulate it to access unintended files.

---

# **Challenge Description**

> ATAN EMPIRE deployed a new internal management portal for their automotive workshop.
> 

> Everything looked fine during testing… until someone accessed something they probably shouldn’t have.
> 

> Can you uncover what was never meant to be accessible?
> 

---

# Atan: Garage Leak Ctf Writeup Guide

# Step 1 - Explore the Website Normally

The first thing when solving a web challenge is **not attacking immediately**.

Open the website and interact with it like a normal user:

- Visit the homepage
- Click navigation buttons
- Open Contact page
- Inspect page elements

Why?

Because many CTF challenges leave hints in hidden comments, source code, or page structure.

---

# Step 2 - Inspect the Website

Open browser Developer Tools:

```jsx
Right Click -> Inspect
```

Finding home.php

![image.png](/assets/img/posts/atan/image.png)

Finding contact.php

![image.png](/assets/img/posts/atan/image_1.png)

![image.png](/assets/img/posts/atan/image_2.png)

---

# Step 3 - Observe URL Changes

After moving around the website, observe the URL:

![image.png](/assets/img/posts/atan/image_3.png)

This parameter is interesting:

```jsx
?page=
```

Why?

Because applications often use it to dynamically load files.

---

# Step 4 - Test Unexpected Input

Before trying complicated payloads, beginners should test random values.

Replace:

```jsx
?page=home.php
```

with:

```
?page=test
```

Why?

We want to see how the application behaves when unexpected input is supplied.

This are the result after we change:

![image.png](/assets/img/posts/atan/image_4.png)

---

# Step 5 - Test for File Inclusion Behavior

Since `page` appears to load files, we can try giving an unexpected path to understand how filtering behaves.

```jsx
http://chall.aturkreatif.site:5858/?page=https://
```

Why use this?

Instead of loading the page, the application rejects it. This tells us the input is being filtered, which means the parameter is processed internally.

The result will get this.

![image.png](/assets/img/posts/atan/image_5.png)

---

# Step 6 - Follow the Hidden Clues 1

Earlier we saw:

```
<!--old env backup still exists-->
```

and

```
<!--old backup system still enabled-->
```

Those hints suggest backup files may still be accessible.

From hint in inspect user can use to go to path back.txt

The hidden comments repeatedly mentioned backups and old files.

This suggests developers may have left internal notes or forgotten files accessible.

A common file to test is:

```jsx
http://chall.aturkreatif.site:5858/?page=../backup.txt
```

Why use:

```
../
```

Because:

```
../
```

moves one directory backward.

If the application loads files directly, moving backward may allow access to unintended files.

The result reveals developer notes.

![image.png](/assets/img/posts/atan/image_6.png)

The clue came from HINT, which:

H move secret files later

I sanitize page parameter

N archive_atan_9281 may still contain recovery data

T verify encoded values before final deployment

---

# Step 7 - Follow the Hidden Clues 2

Since backup files were mentioned several times, test:

```jsx
http://chall.aturkreatif.site:5858/?page=../.env.backup
```

Why this file?

`.env` files frequently contain:

- passwords
- hidden paths
- internal settings

And backup files are often forgotten after development.

The result reveals:

![image.png](/assets/img/posts/atan/image_7.png)

Now we gain three important clues:

1. A hidden archive exists
2. Secret files are inside `/secret/`
3. Values may be Base64 encoded

---

# Step 8 - Checking Common CTF File Names

At this stage, CTF players often test common filenames because challenge authors sometimes leave decoys or accidentally exposed files:

A common first attempt is:

```jsx
http://chall.aturkreatif.site:5858/?page=../flag.txt
```

The file exists and returns:

![image.png](/assets/img/posts/atan/image_8.png)

After we decode the flag `AK26{dGgxc18xc19mNGszX2ZsNGc=}`:

```jsx
AK26{th1s_1s_f4k3_fl4g}
```

It’s A Fake Flag!

Why suspect it is fake?

Earlier developer notes mentioned:

```
remove fake test flag before finals
```

and:

```
inspect old backup values carefully
```

These hints suggest the challenge contains intentionally misleading files. Instead of stopping here, continue investigating backup files and internal archives.

---

# Step 9 - Access Secret Archive

Earlier we accessed:

```
?page=../.env.backup
```

Inside the file we discovered several interesting values:

```
ARCHIVE_PATH=/secret/
FLAG_ENCODING=double_base64
archive_atan_9281
```

Combining both values gives:

```jsx
/secret/archive_atan_9281.txt
```

After All the clue from the inspect and Hint.

```jsx
http://chall.aturkreatif.site:5858/?page=../secret/archive_atan_9281.txt
```

Opening the file reveals:

![image.png](/assets/img/posts/atan/image_9.png)

The value appears encoded.

The file hinted at double_base64, meaning the value was encoded twice.

```jsx
UVVzeU5udE1aakZmYmpCMFgzTTBiajEwTVhveGJtZGZNVzV3ZFhSOQ==
```

Decode Base64 once:

```jsx
QUsyNntMZjFfbjB0X3M0bj10MXoxbmdfMW5wdXR9
```

Decode Base64 a second time:

```jsx
AK26{Lf1_n0t_s4n=t1z1ng_1nput}
```

#