---
title: "PortSwigger Lab 1 - Authentication - Username Enumeration via Different Responses"
date: 2026-03-01
categories: [PortSwigger, Authentication]
tags: [portswigger, authentication, burp-suite, username-enumeration, brute-force]

image:
  path: /assets/img/posts/auth1/banner.png
  alt: PortSwigger Authentication Lab
---

# Portswigger: Lab 1 - Authentication

---

This lab teaches how to use payloads and determine results based on the response length in the Intruder tab.

---

# **Lab: Username enumeration via different responses**

This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

- [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

---

## Solution

Because of the instruction already mention about brute force attacks, the following steps can be follow:

## Step 1: forward POST /login in the intruder

- Navigate to the login page and submit any invalid username and password.
- In Burp Suite, locate the `POST /login` request in **Proxy > HTTP History**.
- Send the request to **Intruder**.
- Highlight the username value and set it as the payload position:

![image.png](/assets/img/posts/auth1/image.png)

- Select **Sniper** as the attack type.
- Load the provided list of candidate usernames as the payload list.

## Step 2: Enumerate a Valid Username

- Start the Intruder attack.
- After the attack completes, sort the results by the **Length** column.
- Most responses return the message **"Invalid username"** and have similar response lengths.
- One payload produces a different response length and returns **"Incorrect password"**, indicating that the username exists in the application.

![image.png](/assets/img/posts/auth1/image1.png)

- The valid username identified is:

```
ajax
```

![image.png](/assets/img/posts/auth1/image2.png)

![image.png](/assets/img/posts/auth1/image3.png)

## Step 3: Brute-Force the Password

1. Return to the Intruder tab and clear the existing payload position.
2. Replace the username with the valid username discovered earlier:

```
username=ajax&password=§test§
```

![image.png](/assets/img/posts/auth1/image4.png)

- Set the password field as the new payload position.
- Load the provided password wordlist.
- Start the Intruder attack.

## Step 4: Identify the Correct Password

- Review the attack results.
- Most requests return a **200 OK** response.
- One request returns a **302 Redirect** response, indicating a successful login.
- Record the password associated with the 302 response.

![image.png](/assets/img/posts/auth1/image5.png)

## Step 5: Access the Account

- Log in using the discovered credentials.
- After successful authentication, access the user account page.

![image.png](/assets/img/posts/auth1/image6.png)