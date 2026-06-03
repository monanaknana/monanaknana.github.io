---
title: "PortSwigger - Authentication Lab 4 - Username enumeration via subtly different responses"
date: 2026-03-01
categories: [PortSwigger, Authentication]
tags: [portswigger, authentication, burp-suite, username-enumeration, brute-force]

image:
  path: /assets/img/posts/auth4/banner.png
  alt: PortSwigger Authentication Lab
---
# Portswigger: Lab 4 - Authentication

---

This lab is vulnerable to username enumeration through subtle differences in error messages. Although the application displays nearly identical responses for invalid login attempts, a small variation in the error message can be used to identify a valid username. Once a valid username is discovered, a password brute-force attack can be performed.

---

# **Lab: Username enumeration via subtly different responses**

This lab is subtly vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

- [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

---

## Solution

Because of the instruction already mention, the following steps can be follow:

## Step 1: Capture the Login Request

- Submit an invalid username and password on the login page.
- In Burp Suite, locate the `POST /login` request.
- Send the request to **Intruder**.
- Highlight the username parameter and verify that it is marked as the payload position:

```
username=§test§&password=test
```

- Select **Sniper** as the attack type.
- Load the provided list of candidate usernames.

## Step 2: Configure Grep - Extract

- Navigate to the **Settings** tab in Intruder.
- Under **Grep - Extract**, click **Add**.
- Scroll through the response until you find the error message:

```
Invalid username or password.
```

- Highlight the message text and click **OK**.
- Burp will automatically create an extraction rule to capture the error message from each response.

![image.png](/assets/img/posts/auth4/image.png)

## Step 3: Identify a Valid Username

- Start the Intruder attack.
- After the attack completes, observe the newly created extraction column.
- Sort the results using this column.
- Most responses contain:

```
Invalid username or password.
```

- One response contains a subtle variation where the message ends with a trailing space instead of a full stop.

This difference indicates a valid username.

![image.png](/assets/img/posts/auth4/image1.png)

The identified username is:

because of the “pi” warnings did not have “.” at the end.

```jsx
username: pi
```

## Step 4: Brute-Force the Password

- Close the attack window and return to Intruder.
- Replace the username payload position with the discovered username:

```
username=pi&password=§test§
```

- Remove the username wordlist.
- Load the candidate password list.
- Start the attack.

## Step 5: Identify the Correct Password

- Review the results after the attack completes.
- Most responses return a **200 OK** status code.
- One request returns a **302 Redirect** response, indicating successful authentication.

The password associated with the 302 response is:

```
superman
```

![image.png](/assets/img/posts/auth4/image2.png)

![image.png](/assets/img/posts/auth4/image3.png)