# 🕵️ HTB Sherlock: Origins Walkthrough

**Category:** DFIR   
**Difficulty:** Very Easy  
**Status:** Completed!

![origins](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/origins.JPG?raw=true)

Hey! I’m still new to Sherlock challenges, but here’s my walkthrough for **"Origins"** from HackTheBox. This one teaches a lot about brute-force attacks, FTP traffic, and how attackers exfiltrate data. Hope it helps someone out there!

---
## Scenario

A major incident has recently occurred at Forela. Approximately 20 GB of data were stolen from internal s3 buckets and the attackers are now extorting Forela. During the root cause analysis, an FTP server was suspected to be the source of the attack. It was found that this server was also compromised and some data was stolen, leading to further compromises throughout the environment. You are provided with a minimal PCAP file. Your goal is to find evidence of brute force and data exfiltration.

---

## 📁 What’s Given?

We’re given a zip file containing a `.pcap` network capture:

- File: `Origins.zip`
- Password: `hacktheblue`

To get started, I downloaded and extracted the file:

<pre>
> wget -O Origins.zip "&lt;download link&gt;"
> 7z x Origins.zip
</pre>

Then opened it in Wireshark:

<pre>
> wireshark ftp.pcap
</pre>

---

## 🔍 Task 1: What is the attacker's IP address?

Since the scenario mentioned FTP, I filtered only FTP traffic:

<pre>
> ftp
</pre>

Found multiple login attempts from a single IP. Clearly a brute-force attempt.

![task1ori](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task1ori.JPG?raw=true)

**Answer:** `15.206.185.207`

---

## 🗺️ Task 2: Using geolocation data, what city is the attacker from?

I took the attacker's IP and looked it up on [ipinfo.io](https://ipinfo.io).

![task2ori](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task2ori.JPG?raw=true)

**Answer:** `Mumbai`

---

## 🖥️ Task 3: Which FTP application and version was used?

Checked the response banners in FTP traffic. You can see this info in one of the early server replies.

![task3](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task3ori.JPG?raw=true)

**Answer:** `vsFTPd 3.0.5`

---

## 🕒 Task 4: When did the brute force attack start?

Filtered traffic to only show connections from the attacker’s IP:

<pre>
> ip.src == 15.206.185.207 and ftp
</pre>

To view the exact time, I switched to UTC format in Wireshark:  
`View > Time Display Format > UTC Date and Time`

![task4](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task4ori.JPG?raw=true)

**Answer:** `2024-05-03 04:12:54`

---

## 🔐 Task 5: What are the correct FTP credentials?

Searched for “Login successful” and followed the TCP stream to find the successful login.

<pre>
> Analyze --> Follow --> TCP Stream
</pre>

![task5](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task5.JPG?raw=true)

**Answer:** `forela-ftp:ftprocks69$`

---

## 📥 Task 6: What FTP command was used to download files?

Filtered for FTP file transfer activity:

<pre>
> ftp-data
</pre>

Found the file downloads and saw the command used: `RETR`

![task6.0](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task6.0.JPG?raw=true)

![taskori](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/taskori.JPG?raw=true)

**Answer:** `RETR`

---

## 🗝️ Task 7: What’s the SSH password?

Exported the FTP-transferred files:

<pre>
> File --> Export Objects --> FTP-DATA
</pre>

Opened `Maintenance-Notice.pdf` and found the temporary password.

![task7ori](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task7ori.JPG?raw=true)

**Answer:** `**B@ckup2024!**`

---

## ☁️ Task 8: What is the s3 bucket URL for the 2023 archive?
## 📧 Task 9: What internal email was used by the attacker?

Opened the `s3_buckets.txt` file:

<pre>
> cat s3_buckets.txt
</pre>

This one was straightforward. Found the URL mentioned directly. Saw the phishing email address used internally.

![task8,9](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task%208,9.JPG?raw=true)

**Answer:** `https://2023-coldstorage.s3.amazonaws.com`

**Answer:** `archivebackups@forela.co.uk`

---

## 💭 Final Thoughts

Honestly, this was a great intro to FTP brute force and basic traffic analysis using Wireshark. Felt like a mini-investigation hehe. And as someone new to these kinds of challenges, it helped me understand how attackers leave traces that we can follow if we know where to look. Looking forward to doing more challenges like this! 🚀
