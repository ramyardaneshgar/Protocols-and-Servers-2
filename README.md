# Protocols-and-Servers-2
Protocol-layer exploitation using tcpdump, Wireshark, Ettercap, Hydra, and SSH/SCP to demonstrate credential sniffing, MITM attacks, and password cracking across Telnet, POP3, IMAP, and FTP.

By Ramyar Daneshgar 


This lab was simulation of offensive reconnaissance and post-exploitation techniques targeting insecure application-layer protocols. Telnet, FTP, SMTP, POP3, and IMAP remain embedded across legacy infrastructure, often overlooked during security assessments despite their susceptibility to plaintext leakage, weak authentication, and protocol manipulation.

My approach combined packet analysis, adversary-in-the-middle simulations, and brute-force credential enumeration to illustrate how improperly secured communications can compromise the confidentiality, integrity, and availability (CIA) of enterprise systems. Simultaneously, I examined cryptographic mitigations and system-hardening measures to align each finding with remediation best practices.

---

## Task 1: Threat Modeling Protocol-Level Risk

The protocols under examination all operate at Layer 7 of the OSI model. Their design predates modern encryption standards, which leaves them exposed to traffic inspection, session hijacking, and credential theft.

In contrast to the CIA model—focused on protecting assets—adversaries typically operate under a DAD framework: Disclosure (data leakage), Alteration (data tampering), and Destruction (data erasure or service disruption). This inversion helped guide how I assessed each protocol’s exposure surface.

These protocols often lack built-in encryption or integrity verification. Therefore, they must be encapsulated within secure transport layers (e.g., TLS) to be safely used in modern networks. Systems still relying on plaintext variants are inherently vulnerable without compensating controls like segmentation, access controls, or encryption gateways.

---

## Task 2: Capturing POP3 Credentials via Passive Sniffing

To validate the ease of credential capture, I conducted a passive network reconnaissance operation using `tcpdump`. POP3 operates over port 110 by default and transmits credentials in plaintext unless explicitly wrapped with SSL/TLS.

I issued the following:

```bash
sudo tcpdump port 110 -A
```

The `-A` flag renders packet contents in ASCII, which simplifies identification of protocol-level commands and credentials. In this case, I extracted the following from live traffic:

```
USER frank  
PASS D2xc9CgD
```

This type of information disclosure results directly from a lack of transport-layer encryption. The absence of confidentiality guarantees allows any threat actor positioned within the traffic flow—either via port mirroring, passive tapping, or successful lateral movement—to harvest credentials without detection.

Wireshark offered a graphical alternative. Applying the display filter:

```
imap
```

enabled streamlined identification of IMAP-specific traffic, reinforcing the principle that plaintext application-layer protocols are fundamentally insecure without cryptographic wrapping.

<img width="1394" height="1012" alt="image" src="https://github.com/user-attachments/assets/72f13103-6daa-4744-bce8-368eee69eb67" />


**Key Takeaways:**

* Tcpdump filter for Telnet traffic: `port 23`
* Wireshark IMAP filter: `imap`

---

## Task 3: Active Interception Using MITM Techniques

<img width="1400" height="756" alt="image" src="https://github.com/user-attachments/assets/9831d88e-fbc3-4e11-aa74-6cea0c8f3093" />


Man-in-the-middle (MITM) attacks exploit the absence of authentication and integrity controls within communication channels. When a client and server fail to verify each other’s identities or lack encryption, an attacker positioned in-line can silently intercept, modify, or redirect traffic.

I explored two popular MITM toolsets:

1. **Ettercap**, which supports:

   * Text-based interface (TUI)
   * Graphical GTK interface
   * Command-line parameters

2. **Bettercap**, which supports:

   * Interactive CLI shell
   * Scripted CLI execution
   * Web-based UI

These tools automate ARP poisoning and facilitate interception of traffic across protocols such as HTTP, FTP, and SMTP. Without TLS, victims receive no indication of tampering. In real-world environments, this class of attack is often coupled with rogue access points or DNS spoofing to ensure traffic passes through the attacker-controlled node.

**Answers:**

* Ettercap interfaces: `3`
* Bettercap invocation methods: `3`

---

## Task 4: Protocol Hardening Through TLS

To mitigate both passive sniffing and active MITM, organizations must encapsulate legacy protocols within a cryptographic transport layer. TLS (Transport Layer Security), the successor to SSL, provides encryption, integrity validation, and endpoint authentication via digital certificates.

During a TLS handshake:

1. The client initiates communication with a `ClientHello`, advertising supported cipher suites.
2. The server replies with a `ServerHello`, shares its public certificate, and selects a cipher.
3. The client and server derive a shared symmetric session key, typically via Diffie-Hellman or RSA key exchange.
4. Both parties switch to encrypted communication using the derived session key.

This handshake ensures:

* Session confidentiality via symmetric encryption
* Authenticity of the server through certificate validation
* Integrity of data using MACs or AEAD constructions

DNS, historically plaintext, now supports DNS over TLS (DoT), which encapsulates DNS queries in an encrypted session, thwarting traffic analysis and DNS spoofing.

**Answer:**

* DNS over TLS acronym: `DoT`

---

## Task 5: SSH and Secure Remote Operations

Secure Shell (SSH) represents the modern, secure replacement for Telnet. It leverages asymmetric cryptography to establish trust between client and server, and symmetric encryption for efficient secure communication.

### Remote Login

I connected to the target system using:

```bash
ssh mark@10.10.122.11
```

Upon password authentication (`XBtc49AB`), I issued:

```bash
uname -r
```

to determine the system’s kernel version.

**Kernel Release:**

* `5.15.0–119-generic`

### Secure File Transfer

Using SCP, which operates atop SSH, I executed:

```bash
scp mark@10.10.122.11:/home/mark/book.txt .
```

The downloaded file measured 415 KB. This demonstrates how SCP ensures confidentiality and integrity by inheriting SSH's cryptographic protections, making it preferable over unencrypted FTP.

**Answer:**

* File size: `415`

---

## Task 6: Password-Based Attacks via Dictionary Enumeration

Despite advances in authentication technology, password-based login remains a common vector of compromise. I targeted the IMAP service using THC Hydra, a fast, parallelized brute-force tool.

With the target username identified as `lazie`, I launched:

```bash
hydra -l lazie -P /usr/share/wordlists/rockyou.txt MACHINE_IP imap
```

Hydra tested each password in the `rockyou.txt` wordlist against the IMAP server. Upon successful authentication, it returned:

**Answer:**

* Password: `butterfly`

This attack exploited the absence of account lockout, throttling, or 2FA. Hydra’s ability to parallelize password guesses makes it highly effective in low-security environments.

---

## Task 7: Protocol Port Knowledge and Security Best Practices

Understanding default ports is essential when constructing firewall rules, performing enumeration, or interpreting traffic captures. This room reinforced the significance of associating services with their common ports and recognizing their secure variants:

* FTP → 21 / FTPS → 990
* Telnet → 23 / SSH → 22
* HTTP → 80 / HTTPS → 443
* IMAP → 143 / IMAPS → 993
* POP3 → 110 / POP3S → 995

Hydra command-line options were also summarized, emphasizing modularity, verbosity flags, and thread configuration (`-t`) to optimize attacks under specific conditions.

<img width="1584" height="702" alt="image" src="https://github.com/user-attachments/assets/48cacb6c-acb1-408a-87b3-ac6a6f2f43d7" />

<img width="1400" height="1046" alt="image" src="https://github.com/user-attachments/assets/91fe5199-4fcf-41fd-8952-f13054f6005a" />



---

## Lessons Learned

1. **Unencrypted Protocols Expose High-Risk Assets**
   Protocols like POP3, IMAP, and Telnet transmit sensitive data in plaintext by default. Without TLS, any actor with packet capture capabilities can harvest credentials or tamper with session content.

2. **MITM Attacks Exploit Protocol Weaknesses, Not Just Network Topology**
   Lack of origin authentication and message integrity opens the door to silent interception and data alteration. TLS, if correctly implemented and validated, provides strong defense against these threats.

3. **Credential Reuse and Weak Passwords Remain a Primary Entry Vector**
   Even in 2025, dictionary-based attacks remain viable due to human behavior. The prevalence of weak or recycled passwords reinforces the need for password policies, multi-factor authentication, and anomaly detection.

4. **Cryptographic Controls Must Be Verified, Not Assumed**
   TLS mitigations rely on proper certificate issuance, trust chain validation, and updated cipher configurations. Misconfigured or expired certificates offer a false sense of security and can even facilitate MITM via downgrade attacks.

5. **Security is Layered, and No Protocol is Inherently Secure**
   SSH is only secure if the host key is verified. TLS only prevents MITM if the certificate is valid and the client performs validation. Passwords only work if paired with rate-limiting and proper entropy. Each layer must be evaluated, enforced, and monitored.

