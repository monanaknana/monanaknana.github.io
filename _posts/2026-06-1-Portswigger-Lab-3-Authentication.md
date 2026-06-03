---
title: "PortSwigger - Authentication Lab 3 - Password reset broken logic"
date: 2026-03-01
categories: [PortSwigger, Authentication]
tags: [portswigger, authentication, burp-suite, username-enumeration, brute-force]

image:
  path: /assets/img/posts/auth3/banner.png
  alt: PortSwigger Authentication Lab
---

# Portswigger: Lab 3 - Authentication

---

This lab demonstrates a flaw in the password reset mechanism. Although a password reset token is generated and sent via email, the application fails to validate the token when the new password is submitted. As a result, an attacker can reset another user's password by modifying the username parameter.

---

# **Lab: Password reset broken logic**

This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.

- Your credentials: `wiener:peter`
- Victim's username: `carlos`

---

## Solution

Because of the instruction already mention, the following steps can be follow:

## Step 1: Request a Password Reset

- Click the **Forgot your password?** link on the login page.
- Enter your own username and submit the request.
- Open the **Email Client** and access the password reset email.

![image.png](/assets/img/posts/auth3/image.png)

- Click the password reset link and set a new password.

```jsx
password: avu
```

![image.png](/assets/img/posts/auth3/image1.png)

## Step 2: Analyze the Password Reset Request

- In Burp Suite, navigate to **Proxy > HTTP History**.
- Locate the password reset request:

```
POST /forgot-password?temp-forgot-password-token=XXXXXXXX
```

![image.png](/assets/img/posts/auth3/image2.png)

## Step 3: Verify Whether the Token Is Validated

- In Burp Repeater, remove the value of the `temp-forgot-password-token` parameter from:
    - The URL
    - The request body (if present)
- Keep the username as your own account.
- Send the modified request.
- Notice that the server still returns a successful response and updates the password.

This confirms that the application does not properly validate the password reset token.

![image.png](/assets/img/posts/auth3/image3.png)

## Step 4: Reset the Victim's Password

- Request another password reset for your account.
- Capture the new password reset request and send it to Burp Repeater.
- Remove the token value again.
- Modify the username parameter from:

```
username=wiener
```

to:

```
username=carlos
```

- Set a new password of your choice:

```
new-password-1=lili
new-password-2=lili
```

- Send the request.
- The application processes the request successfully and updates Carlos's password.

![image.png](/assets/img/posts/auth3/image6.png)

## Step 5: Access Carlos's Account

- Return to the login page.
- Log in using:

```
Username: carlos
Password: lili
```

- Navigate to **My Account**.

![image.png](/assets/img/posts/auth3/image4.png)

![image.png](/assets/img/posts/auth3/image5.png)