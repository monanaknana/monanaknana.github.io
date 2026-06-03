---
title: "PortSwigger - Authentication Lab 5 - Username enumeration via response timing"
date: 2026-03-01
categories: [PortSwigger, Authentication]
tags: [portswigger, authentication, burp-suite, username-enumeration, brute-force]

image:
  path: /assets/img/posts/auth5/banner.png
  alt: PortSwigger Authentication Lab
---
# Portswigger: Lab 5 - Authentication

---

This lab is vulnerable to username enumeration through response timing. Although the application implements IP-based brute-force protection, it trusts the `X-Forwarded-For` header, allowing an attacker to spoof different IP addresses and bypass the rate limit. By measuring response times, a valid username can be identified before performing a password brute-force attack.

---

# **Lab: Username enumeration via response timing**

This lab is vulnerable to username enumeration using its response times. To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

- Your credentials: `wiener:peter`
- [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

---

## Solution

Because of the instruction already mention, the following steps can be follow:

## Step 1: Identify the Rate Limiting Mechanism

- Submit several invalid login attempts.
- Observe that the application eventually displays the message:

```
You have made too many incorrect login attempts. Please try again in 30 minute(s).
```

- Send the `POST /login` request to **Burp Repeater**.
- Add the following header:

```
X-Forwarded-For: 1
```

- Change the value repeatedly to spoof different IP addresses and bypass the login restriction.

*sorry I did not put the screen short of the “X-Forwarded-For: 1” but it need to add at intruder part

![image.png](/assets/img/posts/auth5/image.png)

![image.png](/assets/img/posts/auth5/image1.png)

## Step 2: Identify a Valid Username Using Response Timing

- In Burp Repeater, test different usernames while using a very long password (approximately 100 characters).
- Observe the response times:
    - Invalid usernames return responses with similar timings.
    - Valid usernames take noticeably longer because the application performs additional password validation.
- Send the request to **Burp Intruder**.
- Select **Pitchfork** as the attack type.
- Add the following payload positions:

```
X-Forwarded-For: §1§

username=§wiener§&password=peterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeterpeter
```

- Configure Payload 1:
    - Type: Numbers
    - Range: 1–100
    - Step: 1
- Configure Payload 2:
    - Candidate usernames list
- Start the attack.

![image.png](/assets/img/posts/auth5/image2.png)

![image.png](/assets/img/posts/auth5/image3.png)

## Step 3: Analyze Response Times

- After the attack completes, enable the following columns:
    - Response received
    - Response completed
- Sort the results by response time.
- One username consistently produces a noticeably longer response time than the others.

The identified username is:

```
adm
```

This indicates that the username exists in the application.

![image.png](/assets/img/posts/auth5/image4.png)

## Step 4: Brute-Force the Password

- Create a new Intruder attack.
- Configure the request as follows:

```
X-Forwarded-For: §ip§

username=adm&password=§password§
```

- Select **Pitchfork** attack.
- Configure Payload 1:
    - Numbers list (1–100)
- Configure Payload 2:
    - Candidate password list
- Start the attack.

![image.png](/assets/img/posts/auth5/image5.png)

## Step 5: Identify the Correct Password

- Review the attack results.
- Most requests return **200 OK** responses.
- One request returns a **302 Redirect**, indicating successful authentication.

The identified password is:

```
matthew
```

## Step 6: Access the Account

- Log in using the discovered credentials:

```
Username: adm
Password: matthew
```

- Navigate to **My Account**.

![image.png](/assets/img/posts/auth5/image6.png)