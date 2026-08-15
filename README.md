# Cybersecurity-Lab-Writeups
Cybersecurity Home Lab & Penetration Testing Write-Up

🛡️ Project Overview
This repository documents the creation and execution of a localized, isolated cybersecurity home lab. The objective of this project was to safely practice offensive security methodologies (Red Teaming) while observing and understanding system defenses (Blue Teaming).

Lab Environment:
Attacker Machine: Kali Linux
Web Target: Ubuntu Server running Damn Vulnerable Web Application (DVWA)
Corporate Desktop Target: Windows 10 (Custom Configuration)
Network: VMware/VirtualBox Host-Only Virtual Network (Air-gapped)

<img width="985" height="738" alt="Screenshot 2026-08-14 230440" src="https://github.com/user-attachments/assets/03b72aaa-90d6-4b1c-9028-4a9bcd816d6b" />


🌐 Phase 1: Web Application Testing
The initial phase focused on exploiting common web application vulnerabilities using DVWA.
Command Injection: Successfully injected arbitrary system commands into a web application input field, allowing for direct communication with the underlying Linux operating system.
SQL Injection (SQLi): Exploited a vulnerable database query by injecting boolean logic (' OR '1'='1) into a user ID field. This bypassed the application's authentication logic and successfully dumped all registered user credentials from the backend database.

🔍 Phase 2: Network Reconnaissance
Transitioning to the corporate Windows target, I utilized nmap to map the attack surface and identify potential entry points.
Scan Executed: nmap -sV -O [Target IP]
Findings:
Identified standard Windows networking ports, including Port 445 (SMB) and Ports 135/139 (RPC/NetBIOS).
Successfully fingerprinting the target operating system as Windows 10.
Blue Team Discovery: Discovered enterprise SIEM software (Splunk) running on ports 8000 and 8089, indicating active security monitoring on the target.

<img width="681" height="543" alt="Screenshot 2026-08-14 231409" src="https://github.com/user-attachments/assets/84e0cdbd-4300-4c2a-ab1d-4016d68591dc" />

⚔️ Phase 3: Exploitation & Troubleshooting
This phase focused on attempting to compromise the Windows 10 target via the SMB service using the Metasploit Framework.
Attack Vector 1: SMB Credential Brute-Forcing (Failed Securely)
I utilized the auxiliary/scanner/smb/smb_login module in Metasploit to attempt a dictionary attack against the local Administrator account.
Troubleshooting: Resolved strict Linux file path validation errors by correcting typos in the wordlist directory path (/usr/share/wordlists/fasttrack.txt).
Blue Team Defense Encountered: The attack successfully launched but was halted by the Windows Account Lockout Threshold. The system detected the rapid failed login attempts and temporarily locked the Administrator account, successfully thwarting the brute-force attack.

<img width="991" height="429" alt="Screenshot 2026-08-15 182428" src="https://github.com/user-attachments/assets/9bc0276f-3e40-41ab-8480-cf5bf5e85203" />

Attack Vector 2: Client-Side Attack & Reverse Shell (Successful)
Pivoting from the locked front door, I engineered a client-side attack to bypass network-level restrictions.
Payload Generation: Utilized msfvenom to generate a malicious Windows executable (update.exe) containing a windows/x64/meterpreter/reverse_tcp payload.
Delivery: Hosted the payload on a Python HTTP server (python3 -m http.server 80) running on the Kali attacker machine.
Defense in Depth Encountered: While attempting to download the payload on the Windows target, Microsoft Defender SmartScreen flagged and blocked the file because it could not verify its reputation. I manually bypassed this browser-level defense to execute the file.
Post-Exploitation: The payload successfully connected back to the Metasploit exploit/multi/handler listener, granting a full Meterpreter Reverse Shell.

<img width="374" height="622" alt="Screenshot 2026-08-15 184402" src="https://github.com/user-attachments/assets/16391308-8fa5-4ee3-994c-b577d2709585" />

<img width="1009" height="142" alt="image" src="https://github.com/user-attachments/assets/830143df-5ac8-4a75-b682-035598606449" />

🛠️ Post-Exploitation & Lessons Learned
Once initial access was achieved, I utilized Meterpreter to interact with the system (sysinfo, dropping into a native Windows shell, and capturing desktop screenshots).
Key Takeaways:
Defense in Depth works: Turning off Windows Defender Antivirus was not enough to guarantee a successful attack; Account Lockout policies and SmartScreen provided vital secondary layers of defense.
Attention to detail is critical: Navigating command-line tools requires strict syntax accuracy. Troubleshooting minor path errors (e.g., typing lsd instead of lcd for Local Change Directory) is a fundamental part of the execution process.
Noisy attacks fail in modern environments: Automated, aggressive brute-forcing is easily detected and stopped by modern Active Directory and local security policies. Client-side attacks and social engineering remain highly effective vectors.
