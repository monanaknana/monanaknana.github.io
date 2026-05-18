---
title: "My CTF Writeup"
date: 2026-05-16
categories: [CTF]
tags: [lfi, web]
---

# How to solve if ping is unreachable in linux

I have problem with my ping and unreachable in my kali linux. I looking at internet there no option for me to solve and there only have video to solve my problem. 

first you need to go at below path at your window path:

```jsx
C:\Program Files (x86)\VMware\VMware Workstation
```

Then look for .exe file name vmnetcfg.exe:

![image.png](/assets/img/posts/ping_not_works/image.png)

After that right click that choose run as adminstrator:

![image.png](/assets/img/posts/ping_not_works/image-01.png)

Next u see by default the external connection for type Bridged is automatic.

![image.png](/assets/img/posts/ping_not_works/image-02.png)

Than change it from automatic to your own virtual ethernet adapter, for me is Intel(R) Wi-Fi 6 AX203:

![image.png](/assets/img/posts/ping_not_works/image-03.png)

Than click at Apply and Ok 

Then you get the result will:

![image.png](/assets/img/posts/ping_not_works/image-04.png)