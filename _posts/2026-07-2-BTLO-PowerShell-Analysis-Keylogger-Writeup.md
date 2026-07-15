---
title: "BTLO - PowerShell Analysis: Keylogger Write Up"
date: 2026-07-2
categories:
  - Blue Team Labs Online
  - Write-up
tags:
  - BTLO
  - PowerShell
  - Malware Analysis
  - DFIR

image:
  path: /assets/img/posts/btlo-powershellkeylogger/banner.png
  alt: BTLO PowerShell Analysis Keylogger Challenge Banner

toc: true
comments: true
---

## BTLO - PowerShell Analysis: Keylogger Write Up

> **Challenge:** PowerShell Analysis: Keylogger  
> **Platform:** Blue Team Labs Online (BTLO)  
> **Difficulty:** Easy
> **Category:** Malware Analysis / PowerShell / DFIR

---

Hi gentle readers, in this challenge we'll analyze a suspicious PowerShell script to determine its purpose and extract the information required to answer each question. Instead of executing the script, we'll perform **static analysis**, which allows us to safely inspect the code and understand its behaviour without risking malware execution on our system.

Let's get started. Happy embekk!! 🐐

---

## **Scenario**

A suspicious PowerShell script was found on one of our endpoints. Can you work out what it does?

---

## Setup

### Tools used

- none

### Files provided

The provided evidence consists of a ZIP archive containing another compressed archive, which ultimately reveals a PowerShell script named **HDWallpaperEngine.txt**. Since this script is suspected to be malicious, our analysis will focus on inspecting its contents statically to understand its functionality without executing it.

![image.png](/assets/img/posts/btlo-powershellkeylogger/image.png)

---

## Walkthrough

### Q1: What is the SHA256 hash value for the PowerShell script file?

Run the following PowerShell command:

```jsx
Get-FileHash ".\HDWallpaperEngine.txt" -Algorithm SHA256
```

Why?

- `Get-FileHash` calculates the file hash.
- `Algorithm SHA256` specifies the hashing algorithm required by the challenge.

![image.png](/assets/img/posts/btlo-powershellkeylogger/image1.png)

<aside>
💡

Answer: E0B7A2AD2320AC32C262AEB6FE2C6C0D75449C6E34D0D18A531157C827B9754E

</aside>

---

### Q2: What email address is used to send and receive emails?

Because of this question provide we in .txt file it easy for us to analyze. I just open the file and proceed answer all the question.

![image.png](/assets/img/posts/btlo-powershellkeylogger/image2.png)

<aside>
💡

Answer: chaudhariparth454@gmail.com

</aside>

---

### Q3: What is the password for this email account?

After identifying the email address, the next step is to determine the credentials used to authenticate with the SMTP server.

![image.png](/assets/img/posts/btlo-powershellkeylogger/image3.png)

<aside>
💡

Answer: yjghfdafsd5464562!

</aside>

---

### Q4: What port is used for SMTP?

The malware needs to know which SMTP service port to connect to before sending emails.

![image.png](/assets/img/posts/btlo-powershellkeylogger/image4.png)

<aside>
💡

Answer: 587

</aside>

---

### Q5: What DLL is imported to help record keystrokes?

PowerShell cannot directly capture keyboard events using built-in cmdlets. Instead, the script imports Windows API functions from a Dynamic Link Library (DLL) using `DllImport`.

![image.png](/assets/img/posts/btlo-powershellkeylogger/image5.png)

<aside>
💡

Answer: user32.dll

</aside>

---

### Q6: What directory is the generated txt file put in?

The final step is to determine where the keylogger stores the captured keystrokes before they are sent via email.

![image.png](/assets/img/posts/btlo-powershellkeylogger/image6.png)

<aside>
💡

Answer: temp

</aside>

---

## 🎉 That's it - you've completed **PowerShell Analysis: Keylogger!!**

![image.png](/assets/img/posts/btlo-powershellkeylogger/image7.png)

I hope this write up made the challenge easier to follow and helped you understand where each answer came from instead of just copying them.

Thanks for reading, and I'll see you in the next BTLO write up. Happy embekk! 🐐😆