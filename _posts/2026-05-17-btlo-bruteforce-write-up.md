---
title: "BTLO - Bruteforce Write Up"
date: 2026-06-15
categories:
  - Blue Team Labs Online
  - Write-up
tags:
  - BTLO
  - Splunk
  - Windows Event Logs
  - DFIR

image:
  path: /assets/img/posts/btlo-bruteforce/banner.png
  alt: BTLO Bruteforce Challenge Banner

toc: true
comments: true
---

# BTLO - Bruteforce Write Up

> **Challenge:** Bruteforce  
> **Platform:** Blue Team Labs Online (BTLO)  
> **Difficulty:** Medium
> **Category:** Log Analysis / Windows Security Events / Splunk

---

Hi gentle readers, I analyzed the provided log file and answered all the related questions using Splunk. Don't worry if you haven't installed Splunk yet, I already have a separate write up showing you how to install and set it up. Trust me, life will be much easier once you have Splunk ready! After you've installed it, you can follow along with my method to solve this challenge step by step. Happy embekk!!

---

# **Scenario**

**Can you analyze logs from an attempted RDP bruteforce attack?**

One of our system administrators identified a large number of Audit Failure events in the Windows Security Event log.

There are a number of different ways to approach the analysis of these logs! Consider the suggested tools, but there are many others out there!

---

# Setup

## Tools used

- flareVM
- splunk - load .cvs file
- ipabuse - ip cheaking

## Files provided

![image.png](/assets/img/posts/btlo-bruteforce/image.png)

---

# Walkthrough

## Q1: How many Audit Failure events are there?

Let's start by counting all Windows Security **Event ID 4625** logs, which represent failed logon attempts. 

```jsx
host="BTLO" index=* "Event ID"=4625
| stats count AS "Total Failed Logon Events"
```

We can use a simple Splunk query to get the total number of failed authentication events.

![image.png](/assets/img/posts/btlo-bruteforce/image1.png)

<aside>
💡

Answer: **3103**

</aside>

## Q2: What is the username of the local account that is being targeted?

Now that we know there was a brute-force attack, the next step is identifying which account the attacker was trying to access. Looking at the Event 4625 logs, we can find the username in the **TargetUserName** field. the question provide .envt file - you can open it and it will display like below:

![image.png](/assets/img/posts/btlo-bruteforce/image2.png)

<aside>
💡

Answer: **administrator**

</aside>

---

## Q3: What is the failure reason related to the Audit Failure logs?

Let's inspect one of the failed logon events. The **Failure Reason** field explains why Windows rejected the authentication request.

![image.png](/assets/img/posts/btlo-bruteforce/image3.png)

<aside>
💡

Answer: **Unknown user name or bad password.**

</aside>

---

## Q4: What is the Windows Event ID associated with these logon failures?

An **Event ID** is a unique number used to identify a Windows event. These are some of event ID, to get you an overview:

| Event ID | Description |
| --- | --- |
| **4624** | Successful logon. |
| **4625** | **Failed logon attempt.** |
| **4688** | A new process was created. |
| **4720** | A new user account was created. |
| **4726** | A user account was deleted. |
| **4732** | A user was added to a local group. |
| **4740** | A user account was locked out. |

In this challenge, we'll focus on **Event ID 4625**, which records failed logon attempts.

<aside>
💡

Answer: **4625**

</aside>

---

## Q5: What is the source IP conducting this attack?

Finally, let's check which source ports were used during the attack. We can extract the port numbers from the logs and use Splunk to find both the minimum and maximum values.

```jsx
host="BTLO" index=*("Event ID"=4624 OR "Event ID"=4625)
| table _time "Event ID" 
| sort "Source Network Address" _time
```

![image.png](/assets/img/posts/btlo-bruteforce/image4.png)

![image.png](/assets/img/posts/btlo-bruteforce/image5.png)

<aside>
💡

Answer: **113.161.192.227**

</aside>

---

## Q6: What country is this IP address associated with?

After identifying the attacker's IP address, we can investigate it using an IP reputation service such as IPAbuse, AbuseIPDB, or Virustotal to determine its geographic location. 

For me I’m using IPabuse

![image.png](/assets/img/posts/btlo-bruteforce/image6.png)

<aside>
💡

Answer: **Viet Nam**

</aside>

## Q6: What is the range of source ports that were used by the attacker to make these login requests?

Finally, let's check which source ports were used during the attack. 

```jsx
host="BTLO" index=* "Event ID"=4625
| rex "Source Port:\s+(?<SourcePort>\d+)"
| stats min(SourcePort) as Min_Port max(SourcePort) as Max_Port
```

We can extract the port numbers from the logs and use Splunk to find both the minimum and maximum values.

![image.png](/assets/img/posts/btlo-bruteforce/image7.png)

<aside>
💡

Answer: 49162-65534

</aside>

---

# 🎉 That's it - you've completed **Bruteforce**!

![image.png](/assets/img/posts/btlo-bruteforce/image8.png)

I hope this write up made the challenge easier to follow and helped you understand where each answer came from instead of just copying them.

Thanks for reading, and I'll see you in the next BTLO write up. Happy embekk! 🐐😆