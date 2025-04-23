# 🕵️ HTB Sherlock: Origins Walkthrough

**Category:** Forensics  
**Difficulty:** Very Easy  
**Status:** Completed!

Hey! I’m still new to Sherlock challenges, but here’s my walkthrough for **"Origins"** from HackTheBox. This one teaches a lot about brute-force attacks, FTP traffic, and how attackers exfiltrate data. Hope it helps someone out there!

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

📸 *[Insert screenshot]*  
**Answer:** `15.206.185.207`

---

## 🗺️ Task 2: Using geolocation data, what city is the attacker from?

I took the attacker's IP and looked it up on [ipinfo.io](https://ipinfo.io).

📸 *[Insert screenshot]*  
**Answer:** `Mumbai`

---

## 🖥️ Task 3: Which FTP application and version was used?

Checked the response banners in FTP traffic. You can see this info in one of the early server replies.

📸 *[Insert screenshot]*  
**Answer:** `vsFTPd 3.0.5`

---

## 🕒 Task 4: When did the brute force attack start?

Filtered traffic to only show connections from the attacker’s IP:

<pre>
> ip.src == 15.206.185.207 and ftp
</pre>

To view the exact time, I switched to UTC format in Wireshark:  
`View > Time Display Format > UTC Date and Time`

📸 *[Insert screenshot]*  
**Answer:** `2024-05-03 04:12:54`

---

## 🔐 Task 5: What are the correct FTP credentials?

Searched for “Login successful” and followed the TCP stream to find the successful login.

<pre>
> Analyze --> Follow --> TCP Stream
</pre>

📸 *[Insert screenshot]*  
**Answer:** `forela-ftp:ftprocks69$`

---

## 📥 Task 6: What FTP command was used to download files?

Filtered for FTP file transfer activity:

<pre>
> ftp-data
</pre>

Found the file downloads and saw the command used: `RETR`

📸 *[Insert screenshot]*  
**Answer:** `RETR`

---

## 🗝️ Task 7: What’s the SSH password?

Exported the FTP-transferred files:

<pre>
> File --> Export Objects --> FTP-DATA
</pre>

Opened `Maintenance-Notice.pdf` and found the temporary password.

📸 *[Insert screenshot]*  
**Answer:** `**B@ckup2024!**`

---

## ☁️ Task 8: What is the s3 bucket URL for the 2023 archive?

This one was straightforward. Found the URL mentioned directly.

📸 *[Insert screenshot]*  
**Answer:** `https://2023-coldstorage.s3.amazonaws.com`

---

## 📧 Task 9: What internal email was used by the attacker?

Opened the `s3_buckets.txt` file:

<pre>
> cat s3_buckets.txt
</pre>

Saw the phishing email address used internally.

📸 *[Insert screenshot]*  
**Answer:** `archivebackups@forela.co.uk`

---

## 💭 Final Thoughts

Honestly, this was a great intro to FTP brute force and basic traffic analysis using Wireshark. Felt like a mini-investigation hehe. And as someone new to these kinds of challenges, it helped me understand how attackers leave traces that we can follow if we know where to look. Looking forward to doing more challenges like this! 🚀
