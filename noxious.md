# 🕵️ HTB Sherlock: Noxious Walkthrough

**Category:** SOC  
**Difficulty:** Very Easy  
**Status:** Completed!

![noxious](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/noxious.JPG?raw=true)
---

## Scenario

The IDS device alerted us to a possible rogue device in the internal Active Directory network. The Intrusion Detection System also indicated signs of LLMNR traffic, which is unusual. It is suspected that an LLMNR poisoning attack occurred. The LLMNR traffic was directed towards Forela-WKstn002, which has the IP address 172.17.79.136. A limited packet capture from the surrounding time is provided to you, our Network Forensics expert. Since this occurred in the Active Directory VLAN, it is suggested that we perform network threat hunting with the Active Directory attack vector in mind, specifically focusing on LLMNR poisoning.

---

## 📁 What’s Given?

We’re given a zip file containing a `.pcap` network capture:

- File: `noxious.zip`
- Password: `hacktheblue`

---

### 🔍Task 1: Find the Malicious IP Address

The security team suspects a rogue device running the Responder tool to perform an LLMNR poisoning attack. We need to find the malicious IP address.
Use this filter: 
<pre> > udp.port == 5355 </pre>

![task1nox](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task1nox.JPG?raw=true)

172.17.79.136 initiated queries for “DCC01,” which was actually a typo, and 172.17.79.135 responded to it.

**Answer**: 172.17.79.135

---

### 🔍Task 2: Find the Hostname of the Rogue Machine

Now, let's identify the hostname of the rogue machine.

<pre> > ip.addr == 172.17.79.135 && dhcp </pre>

![task2nox](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task2nox.JPG?raw=true)

We can see the hostname in the DHCP packet.

**Answer**: kali

---

### 🔍Task 3: Identify the Username Whose Hash Was Captured

Let's confirm if the attacker captured the user's hash. We need to find the username.

<pre> > ntlmssp </pre>

![task3,44nox](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task3,44nox.JPG?raw=true)

The username is in the NTLMSSP_AUTH packet.

**Answer**: john.deacon

---

### 🔍Task 4: Find the Time the Hash Was First Captured

When was the user's hash first captured? Let's check the NTLM traffic.

Go to <pre> > View --> Time display format --> UTC DATE</pre>
- **Filter**: <pre> ntlmssp </pre>

We look at the first 3 packets (from NTLMSSP_NEGOTIATE to NTLMSSP_AUTH).

**Answer**: 2024-06-24 11:18:30

---

### 🔍Task 5: What Was the Typo Made by the Victim?

When navigating to the file share, what typo caused the victim’s credentials to be leaked?

The victim typed “DCC01” instead of “DC01,” triggering the fallback to LLMNR, which allowed the attacker to respond as the domain controller.

**Answer**: DCC01

---

### 🔍Task 6: Find the NTLM Server Challenge Value

Now let’s look for the NTLM server challenge value.

- **Filter**: <pre> ntlmssp </pre>

Expand to find the NTLM Server Challenge.

![task6](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task6.JPG?raw=true)

**Answer**: 601019d191f054f1

---

### 🔍Task 7: Find the NTProofStr Value

Next, we need to find the NTProofStr value.

In the NTLM response packet, we can find the NTProofStr.

![task7](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task7.JPG?raw=true)

**Answer**: c0cc803a6d9fb5a9082253a04dbd4cd4

---

### 🔍Task 8: Recover the Victim's Password

Let's use the values we’ve found to crack the victim’s password.

- **Create file with these values**:
  
<pre> john.deacon::FORELA:601019d191f054f1:c0cc803a6d9fb5a9082253a04dbd4cd4:010100000000000080e4d59406c6da01cc3dcfc0de9b5f2600000000020008004e0042004600590001001e00570049004e002d00360036004100530035004c003100470052005700540004003400570049004e002d00360036004100530035004c00310047005200570054002e004e004200460059002e004c004f00430041004c00030014004e004200460059002e004c004f00430041004c00050014004e004200460059002e004c004f00430041004c000700080080e4d59406c6da0106000400020000000800300030000000000000000000000000200000eb2ecbc5200a40b89ad5831abf821f4f20a2c7f352283a35600377e1f294f1c90a001000000000000000000000000000000000000900140063006900660073002f00440043004300300031000000000000000000 </pre>


- **Run hashcat**:
<pre> > hashcat -a0 -m5600 hashfile.txt /usr/share/wordlists/rockyou.txt</pre>

**Answer**: NotMyPassword0k?

---

### 🔍Task 9: Find the Actual File Share the Victim Was Trying to Access

Finally, what file share was the victim trying to access?

![task9](https://github.com/candypopZZ/htb-sherlock-walkthrough/blob/main/images/task9.JPG?raw=true)

- **Default File**: `\\DC01.forela.local\IPC$`
- **Non-Default File**: `\\DC01\DC-Confidential`

**Answer**: \\DC01\DC-Confidential

---

### 💭Reflection

This challenge really gave me some good insight into how attackers can leverage tools like Responder for LLMNR poisoning and the ease with which credentials can be captured. It was interesting to see how a simple typo by the victim could result in such an attack, and how important it is to have a strong network defense in place.

