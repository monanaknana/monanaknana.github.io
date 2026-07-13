---
title: "BTLO - Log Analysis: Sysmon Write Up"
date: 2026-06-20
categories:
  - Blue Team Labs Online
  - Write-up
tags:
  - BTLO
  - Splunk
  - Sysmon
  - Windows
  - DFIR
  - Log Analysis

image:
  path: /assets/img/posts/btlo-loganalysis/banner.png
  alt: BTLO Log Analysis Sysmon Challenge Banner

toc: true
comments: true
---

## BTLO - Log Analysis: Sysmon


> **Platform:** Blue Team Labs Online (BTLO)
> **Difficulty:** Medium

---
Hi starsss! 

In this write up, I'll show you how I solved the BTLO Log Analysis: Sysmon challenge using Splunk. If you haven't installed Splunk yet, check out my setup guide first, then come back and follow along step by step.

Happy embekk!! 🐐

---

## **Scenario**

You are provided with Sysmon logs from a compromised endpoint. Analyse the logs to find out the steps and techniques used by the attacker.

---

## Setup

### Tools used

- splunk

### Files provided

![image.png](/assets/img/posts/btlo-loganalysis/image.png)

First, upload the provided Sysmon log into Splunk. I begin analyze the file. here the splunk after I upload:

![image.png](/assets/img/posts/btlo-loganalysis/image1.png)

---

## Walkthrough

### Q1: What is the file that gave access to the attacker?

The keyword here is **"gave access"**, so we need to identify the file that initially executed the attack. Below is the query that I been using to solve this question.

```jsx
host="BTLO" index="btlo_log_analysis_sysmon" "Event.System.EventID"=1
| table Event.EventData.UtcTime Event.EventData.IntegrityLevel Event.EventData.ParentImage Event.EventData.Image Event.EventData.ParentCommandLine
| reverse
```

Since the question asks about the initial access, we search for **Process Create (Event ID 1)** events to identify the first malicious process.

From the process creation events, we can see the attacker executed the following command:

![image.png](/assets/img/posts/btlo-loganalysis/image2.png)

The logs show **mshta.exe** opening **updater.hta**.

Which

HTA files can execute scripts with the user's permissions and are commonly abused to deliver malware.

<aside>
💡

Answer: updater.hta

</aside>

---

### Q2: What is the powershell cmdlet used to download the malware file and what is the port?

By using same query from question 1, will find out below:

![image.png](/assets/img/posts/btlo-loganalysis/image3.png)

from that we know that the cmdlet is `Invoke-WebRequest` , next we need to find the port. I try go the the github using url that in the log. 

![image.png](/assets/img/posts/btlo-loganalysis/image4.png)

After a bit reading, the GitHub page explains that Juicy Potato uses port **6666**, which matches the question. below it the port. yeahhhh

![image.png](/assets/img/posts/btlo-loganalysis/image5.png)

<aside>
💡

Answer: Invoke-WebRequest, 6666

</aside>

---

### Q3: What is the name of the environment variable set by the attacker?

We can reuse the previous query. We can see at time `2021-05-07 12:22:39.500`.

![image.png](/assets/img/posts/btlo-loganalysis/image6.png)

The attacker changes the COMSPEC environment variable to point to supply.exe.

<aside>
💡

Answer: comspec=C:\windows\temp\supply.exe

</aside>

---

### Q4: What is the process used as a LOLBIN to execute malicious commands?

A LOLBIN (Living Off the Land Binary) is a legitimate Windows executable abused by attackers to perform malicious actions.

So the anwser at the image process in the log.

![image.png](/assets/img/posts/btlo-loganalysis/image7.png)

<aside>
💡

Answer: ftp.exe

</aside>

---

## Q5: Malware executed multiple same commands at a time, what is the first command executed?

After attacker change the environment, the first command executed are `ipconfig`.

![image.png](/assets/img/posts/btlo-loganalysis/image8.png)

<aside>
💡

Answer: ipconfig

</aside>

---

## Q6: Looking at the dependency events around the malware, can you able to figure out the language, the malware is written

Now I change the query to find the language.

```jsx
host="BTLO" index="btlo_log_analysis_sysmon" "Event.System.EventID"=*
| table Event.EventData.UtcTime Event.EventData.TargetFilename Event.EventData.IntegrityLevel Event.EventData.ParentImage Event.EventData.Image Event.EventData.ParentCommandLine 
| reverse
```

by using query `Event.EventData.TargetFilename`  we will find the language that malware written are phyton.

![image.png](/assets/img/posts/btlo-loganalysis/image9.png)

<aside>
💡

Answer: python

</aside>

---

### Q7: Malware then downloads a new file, find out the full url of the file download

The command line contains an `Invoke-WebRequest` command with the download URL.

![image.png](/assets/img/posts/btlo-loganalysis/image10.png)

<aside>
💡

Answer: https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe

</aside>

---

### Q8: What is the port the attacker attempts to get reverse shell?

The attacker executes `nc.exe` with the argument `192.168.1.11:9898`, indicating the reverse shell connects to port **9898**.

![image.png](/assets/img/posts/btlo-loganalysis/image11.png)

<aside>
💡

Answer: 9898

</aside>

---


## 🎉 That's it - you've completed Log Analysis: Sysmon!

![image.png](/assets/img/posts/btlo-loganalysis/image12.png)

I hope this write up made the challenge easier to follow and helped you understand where each answer came from instead of just copying them.

Thanks for reading, and I'll see you in the next BTLO write up. Happy embekk! 🐐😆