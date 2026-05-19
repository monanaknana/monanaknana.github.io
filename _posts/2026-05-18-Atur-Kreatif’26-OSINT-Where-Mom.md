---
title: "OSINT - Where'sMom? CTF Writeup Guide"
date: 2026-05-18
categories: [CTF, OSINT]
tags: [OSINT,  atur-kreatif26]

image:
  path: /assets/img/posts/crown/banner.png
  alt: OSINT - Where'sMom? CTF Challenge

---


## Challenge Description

> Your mom said she was just going out for a quick sightseeing trip. A few hours later, she suddenly sent a photo through her friend and mentioned she’d be taking a free bus service around 12:30 PM before disappearing again.

> The only clue you have is the photo itself.

> After digging through local public transport information, you discover that multiple free bus routes operate around the area. However, only one route matches the terminal closest to the location shown in the image.

> Now it’s up to you to trace the location, investigate the nearby public transport system, and identify the correct bus route your mom might have taken.!


Participants are given:

![Where'sMom Challenge](/assets/img/posts/crown/crown.png)

* A photo containing a monument/location
* A description mentioning:

  * a free bus service
  * operating around **12:30 PM**

The objective is to determine the correct bus route.

## Flag Format

```text
AK26{bus_route}
```
---

## Atur Kreatif’26 OSINT - Where'sMom?

### Step 1 – Analyze the Image

The monument shown in the image is distinctive and recognizable.

Using:

* Google Lens
* Manual landmark searching for:

  * `"mahkota johor"`
  * `"Crown in johor"`

These searches will lead to several monuments and landmarks in Johor. From there, participants must narrow down the correct location by comparing the surrounding environment in the image.

After matching the monument and surrounding layout, participants can determine the exact location in the **Pasir Gudang** area, which can be identify from **Tiktok**.

---

### Step 2 – Identify the Nearest Terminal

Once the monument’s location is identified, participants can search for nearby public transport terminals or free bus services operating in the area.

The challenge description specifically mentions:

* a free bus service (https://paj.com.my/)
* operating around **12:30 PM**

![Where'sMom Challenge](/assets/img/posts/crown/bus_route.png)

Using local transport information, participants can identify the correct route associated with the nearest terminal.

---

### Flag

```text
AK26{P-314}
```
