---
title: "BTLO - Paranoid Write Up"
date: 2026-07-1
categories:
  - Blue Team Labs Online
  - Write-up
tags:
  - BTLO
  - Splunk
  - Linux Audit Logs
  - DFIR
  - Privilege Escalation

image:
  path: /assets/img/posts/btlo-paranoid/banner.png
  alt: BTLO Paranoid Challenge Banner

toc: true
comments: true
---

## BTLO - Paranoid Writeup

> **Challenge:** Paranoid  
> **Platform:** Blue Team Labs Online (BTLO)  
> **Difficulty:** Medium  
> **Category:** Linux Audit Logs / DFIR / Splunk

---

Hi gentle readers! 👋

In this write-up, I'll walk through the **Paranoid** challenge from **Blue Team Labs Online (BTLO)** using **Splunk** to investigate Linux audit logs.

If you haven't installed Splunk yet, don't worry! I already have a separate guide that explains how to install and configure Splunk before starting these challenges. Once your environment is ready, you can follow this write-up step by step and compare your investigation with mine.

As always, I won't just give the answers—I’ll explain **why** I used each query and **how** I arrived at the answer, so you can understand the investigation process instead of simply copying the results.

Happy embekk! 🐐😆

---

## **Scenario**

I'm not paranoid, you are. (nothing personal, I just put what the question mention ;))

---

## Setup

### Tools used

- splunk

### Files provided

![image.png](image.png)

The interface after upload the log:

![image.png](image1.png)

First of all, I usually will refer what `type`they have.

![image.png](image2.png)

---

## Walkthrough

### Q1: What account was compromised?

Before investigating the compromised account, we first need to identify what type of logs are available. Since these are Linux audit logs, the `type` field is useful for understanding the activity recorded.

First to know the account that been compromise I using below query:

```jsx
host="BTLO" index="btlo-paranoid-medium" type=USER_AUTH
| table _time acct hostname res terminal
| sort _time
```

I'm using USER_AUTH or u also can use USER_LOGIN. after the u will see account that they success to login after attempt of brute force:

![image.png](image3.png)

<aside>
💡

Answer: btlo

</aside>

---

### Q2: What attack type was used to gain initial access?

Using same query will already can verify that attacker gain initial access thru brute force.

![image.png](image4.png)

Multiple consecutive failed SSH authentication attempts followed by a successful login from the same source IP indicate a brute-force attack.

<aside>
💡

Answer: Brute Force

</aside>

---

### Q3: What is the attacker's IP address?

![image.png](image5.png)

From the USER_AUTH logs, all failed login attempts and the eventual successful authentication originated from `192.168.4.155`. Therefore, this is the attacker's IP address.

<aside>
💡

Answer: 192.168.4.155

</aside>

---

### Q4: What tool was used to perform system enumeration?

![image.png](image6.png)

Since the attacker successfully authenticated at **08:23:13**, I limited my search to events occurring after this timestamp. I reviewed all `EXECVE` events to identify commands executed by the attacker.

Why? 
The `EXECVE` audit type records every executed command. Since enumeration involves running many commands, this is the best place to investigate attacker activity.

```jsx
host="BTLO" index="btlo-paranoid-medium" type=EXECVE
| where _time >= strptime("2021-10-05 08:23:00","%Y-%m-%d %H:%M:%S")
| table _time a0 a1 a2
| reverse
```

Initially, I observed common enumeration commands such as `hostname`, `whoami`, `sudo -l`, and `uname`. 

![image.png](image7.png)

Continuing through the timeline, I noticed a `wget` command downloading a shell script, which was later identified as **LinPEAS**.

![image.png](image8.png)

I then expanded the event details and observed that the attacker downloaded `linpeas.sh`, confirming that LinPEAS was used for system enumeration.

![image.png](image9.png)

<aside>
💡

Answer: linpeas

</aside>

---

### Q5: What is the name of the binary and pid used to gain root?

To reduce the amount of events that needed to be reviewed, I first searched for `USER_START` events. This audit type records the start of a user session. Since I was looking for the process that granted root access, I used these events to identify when a new root session was established after the attacker's compromise.

```jsx
host="BTLO" index="btlo-paranoid-medium" type=USER_START
| table _time acct
```

![image.png](image10.png)

Although there are several `root` sessions in the logs, they occurred **before** the attacker successfully logged in at **08:23:13** and are likely normal system activity. After the attacker gained access, I observed another `root` session at **08:27:14**, which gave me a narrower timeframe to investigate.

I then pivoted to the `SYSCALL` events around that timestamp and filtered for processes that were executed with root privileges (`uid=0`) but originated from a non-root authenticated user (`auid!=0`). This helps identify potential privilege escalation activity.

```jsx
host="BTLO" index="btlo-paranoid-medium" type=SYSCALL
| where uid=0 AND auid!=0
| table _time pid exe uid auid comm success
| sort _time
```

![image.png](image11.png)

![image.png](image12.png)

From the results, I observed that a **root** session started at `08:27:14`, indicating that privilege escalation occurred around this time.

From the results, I identified **PID 829992** associated with `/usr/bin/sudo`. To determine the actual exploit binary, I reviewed the corresponding `EXECVE` events.

```jsx
host="BTLO" index="btlo-paranoid-medium" type=SYSCALL
| where uid=0 AND auid!=0
| table _time pid exe uid auid comm success
| sort _time
```

![image.png](image13.png)

```jsx
host="BTLO" index="btlo-paranoid-medium" type=EXECVE
| where _time >= strptime("2021-10-05 08:27:00","%Y-%m-%d %H:%M:%S")
| table _time a0 a1 a2
| reverse
```

![image.png](image14.png)

The results show the execution of `./evil`, which is the exploit binary used to abuse the vulnerable version of `sudo`

To understand:

- USER_START → Reduce the investigation scope by identifying when the new root session appeared.
- SYSCALL → Find the PID and privileged process.
- EXECVE → Confirm the exploit binary.

<aside>
💡

Answer: evil, 829992

</aside>

---

### Q6: What CVE was exploited to gain root access? (Do your research!)

I search: `sudo escilaton evil CVE exploitation to gain root access` 

The first from google cloud

![image.png](image15.png)

<aside>
💡

Answer: **CVE-2021-3156**

</aside>

---

### Q7: What type of vulnerability is this?

Searching for the observed sudo exploit revealed CVE-2021-3156, commonly known as Baron Samedit, a heap-based buffer overflow vulnerability affecting sudo. 

<aside>
💡

Answer: Heap_Based Buffer Overflow

</aside>

---

### Q8: What file was exfiltrated once root was gained?

```jsx
host="BTLO" index="btlo-paranoid-medium" type=EXECVE
| where _time >= strptime("2021-10-05 08:27:00","%Y-%m-%d %H:%M:%S")
| table _time a0 a1 a2
| reverse
```

![image.png](image16.png)

<aside>
💡

Answer: shadow

</aside>

---

---

## 🎉 That's it - you've completed Paranoid!

![image.png](image17.png)

I hope this write up made the challenge easier to follow and helped you understand where each answer came from instead of just copying them.

Thanks for reading, and I'll see you in the next BTLO write up. Happy embekk! 🐐😆