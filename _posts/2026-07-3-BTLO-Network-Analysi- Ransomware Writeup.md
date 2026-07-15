---
title: "BTLO - Network Analysis: Ransomware Write Up"
date: 2026-07-3
categories:
  - Blue Team Labs Online
  - Write-up
tags:
  - BTLO
  - Wireshark
  - Network Forensics
  - Malware Analysis
  - DFIR

image:
  path: /assets/img/posts/btlo-ransomware/banner.png
  alt: BTLO Network Analysis Ransomware Challenge Banner

toc: true
comments: true
---

## BTLO - Network Analysis: Ransomware Write Up

> **Challenge:** Network Analysis: Ransomware  
> **Platform:** Blue Team Labs Online (BTLO)  
> **Difficulty:** Medium  
> **Category:** Network Forensics / Malware Analysis / DFIR

---

Hi gentle readers, in this challenge we'll investigate a ransomware infection using the provided packet capture (PCAP), ransom note, and encrypted document. Throughout this write-up, we'll analyze the network traffic to identify how the malware was delivered, determine the ransomware family, extract valuable Indicators of Compromise (IOCs), and recover the encrypted file.

This challenge is a great introduction to network forensics, malware analysis, and ransomware investigations using Wireshark. Let's get started. Happy embekk!! 🐐

---

## **Scenario**

ABC Industries worked day and night for a month to prepare a tender document for a prestigious project that would secure the company’s financial future. The company was hit by ransomware, believed to be conducted by a competitor, and the final version of the tender document was encrypted. Right now they are in need of an expert who can decrypt this critical document. All we have is the network traffic, the ransom note, and the encrypted ender document. Do your thing Defender!

---

## Setup

### Tools used

- Python
- Wireshark

### Files provided

![image.png](/assets/img/posts/btlo-ransomware/image.png)

Inside folder Challenge Files:

![image.png](/assets/img/posts/btlo-ransomware/image1.png)

these are the ransom note:

![image.png](/assets/img/posts/btlo-ransomware/image2.png)

---

## Walkthrough

Before answering the challenge questions, I first examined the provided evidence, which consisted of a packet capture (PCAP), an encrypted PDF document, and the ransom note. The investigation focused on identifying how the ransomware was delivered, determining the malware family, extracting indicators of compromise (IOCs), and ultimately recovering the encrypted file.

### Q1: What is the operating system of the host from which the network traffic was captured?

To identify the operating system used during packet capture, open:

Statistics → Capture File Properties.

![image.png](/assets/img/posts/btlo-ransomware/image3.png)

<aside>
💡

Answer: 32-bit Windows 7 Service Pack 1, build 7601

</aside>

---

### Q2: What is the full URL from which the ransomware executable was downloaded?

Since the question asks where the ransomware was downloaded from, we need to locate the `HTTP` request responsible for downloading the executable

![image.png](/assets/img/posts/btlo-ransomware/image4.png)

By filtering the HTTP traffic, I identified a GET request downloading `safecrypt.exe` from an internal web server. This indicates the ransomware executable was delivered over HTTP.

![image.png](/assets/img/posts/btlo-ransomware/image5.png)

<aside>
💡

Answer: http://10.0.2.15:8000/safecrypt.exe

</aside>

---

### Q3: Name the ransomware executable file?

From the same HTTP GET request identified in the previous question, the requested filename is clearly shown as **safecrypt.exe**.

![image.png](/assets/img/posts/btlo-ransomware/image6.png)

 Since this is the executable downloaded by the victim machine, it is the ransomware binary.

<aside>
💡

Answer: safecrypt.exe

</aside>

---

### Q4: What is the MD5 hash of the ransomware?

Since the ransomware executable was transferred over HTTP, Wireshark allows us to extract it directly using:

File → Export Objects → HTTP

![image.png](/assets/img/posts/btlo-ransomware/image19.png)

Then, select the row → save.

![image.png](/assets/img/posts/btlo-ransomware/image7.png)

After exporting the file, we can calculate its MD5 hash using the `md5sum` command.

![image.png](/assets/img/posts/btlo-ransomware/image8.png)

<aside>
💡

Answer: 4a1d88603b1007825a9c6b36d1e5de44

</aside>

---

### Q5: What is the name of the ransomware?

Using the MD5 hash obtained in the previous step, I searched the hash on VirusTotal. Than go to COMMUNITY panel

![image.png](/assets/img/posts/btlo-ransomware/image9.png)

VirusTotal correlates malware hashes with detections from multiple antivirus vendors. The results identify the sample as **TeslaCrypt**, confirming the ransomware family.

![image.png](/assets/img/posts/btlo-ransomware/image10.png)

<aside>
💡

Answer: TeslaCrypt

</aside>

---

### Q6: What is the encryption algorithm used by the ransomware, according to the ransom note?

The ransom note explains how the victim's files were encrypted. Reading the note shows the statement:

![image.png](/assets/img/posts/btlo-ransomware/image11.png)

Therefore, the ransomware uses **RSA-4096** as stated by the attackers.

<aside>
💡

Answer: RSA-4096

</aside>

---

### Q7: What is the domain beginning with ‘d’ that is related to ransomware traffic?

Since the question asks for a domain beginning with the letter **d**, we can inspect the DNS traffic generated during the ransomware infection. Applying the display filter:

```jsx
dns.qry.name matches "^d"
```

![image.png](/assets/img/posts/btlo-ransomware/image12.png)

<aside>
💡

Answer: dunyamuzelerimuzesi.com

</aside>

---

### Q8: Decrypt the Tender document and submit the flag

After identifying the ransomware as TeslaCrypt, I found that the ransomware operators publicly released the master private key in 2016 after shutting down their operation.

![image.png](/assets/img/posts/btlo-ransomware/image13.png)

The encrypted file already contains the required **public key** in its header. Using the released master private key together with the file header, I ran the Python decryption script to recover the original document.

![image.png](/assets/img/posts/btlo-ransomware/image14.png)

The script successfully generated the recovered PDF file.

![image.png](/assets/img/posts/btlo-ransomware/image15.png)

below is the recover file:

![image.png](/assets/img/posts/btlo-ransomware/image16.png)

Finally, I opened the recovered PDF to verify that the file had been successfully restored.

![image.png](/assets/img/posts/btlo-ransomware/image17.png)

<aside>
💡

Answer: BTLO-T3nd3r-Fl@g

</aside>

---

---

## 🎉 That's it - you've completed Network Analysis: Ransomware!

![image.png](/assets/img/posts/btlo-ransomware/image18.png)

I hope this write up made the challenge easier to follow and helped you understand where each answer came from instead of just copying them.

Thanks for reading, and I'll see you in the next BTLO write up. Happy embekk! 🐐😆