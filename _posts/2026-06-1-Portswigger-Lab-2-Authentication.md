---
title: "PortSwigger - Authentication Lab 2 - 2FA simple bypass"
date: 2026-03-01
categories: [PortSwigger, Authentication]
tags: [portswigger, authentication, burp-suite, username-enumeration, brute-force]

image:
  path: /assets/img/posts/auth2/banner.png
  alt: PortSwigger Authentication Lab
---
# Portswigger: Lab 2 - Authentication

---

This lab demonstrates a flaw in the two-factor authentication implementation. Although the application requires a verification code after login, it fails to properly enforce the 2FA check before granting access to account resources.

---

# **Lab: 2FA simple bypass**

This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.

- Your credentials: `wiener:peter`
- Victim's credentials `carlos:montoya`

---

## Solution

Because of the instruction already mention, the following steps can be follow:

## Step 1: Log In to Your Own Account

- Navigate to the login page.
- Log in using the provided credentials:

```
Username: wiener
Password: peter
```

![image.png](/assets/img/posts/auth2/image.png)

- After logging in, a 2FA verification code is sent to the email client.
- Open the **Email Client** and retrieve the verification code.
- Enter the code to complete authentication.

![image.png](/assets/img/posts/auth2/image1.png)

![image.png](/assets/img/posts/auth2/image2.png)

## Step 2: Identify the Account URL

- After successfully logging in, navigate to the **My Account** page.
- Observe the URL and note the account identifier.

```jsx
https://0ad000f203b7a8878353698600530037.web-security-academy.net/my-account?id=wiener
```

## Step 3: Log Out and Authenticate as the Victim

- Log out of the current account.
- than chnage the url with victim's credential

```jsx
Username: carlos 
Password: montoya
```

- The application redirects to the 2FA verification page.

## Step 4: Bypass the 2FA Process

1. Do not enter a verification code.
2. While on the 2FA verification page, manually change url id from `my-account?id=wiener`  to `my-account?id=carlos`

![image.png](/assets/img/posts/auth2/image3.png)