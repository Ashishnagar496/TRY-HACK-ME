# Hidden Deep Into my Heart

# **Hidden Deep Into my Heart**

## **Objective**

Locate a hidden secret within the web application by discovering concealed directories and exploiting weak credentials.

## **Steps Performed**

1. **Initial Reconnaissance**
    - The room introduction hinted at something hidden deep within the system.
    - The main page displayed a simple message with no interactive elements, suggesting hidden paths.
2. **Robots.txt Discovery**
    - Checked `/robots.txt` based on common practice.
    - Content revealed:
        
        text
        
        ```
        User-agent: *
        Disallow: /cupids_secret_vault/*
        
        # cupid_arrow_2026!!!
        ```
        
    - This disclosed a disallowed directory and a potential password clue.
3. **Exploring the Disclosed Path**
    - Navigated to `http://<IP>:5000/cupids_secret_vault/`
    - Page showed: "Cupid's Secret Vault. You've found the secret vault, but there's more to discover..."
    - Indicated further hidden content within that directory.
4. **Directory Bruteforcing**
    - Used `gobuster` to enumerate subdirectories under `/cupids_secret_vault/`:
        
        bash
        
        ```
        gobuster dir -u http://<IP>:5000/cupids_secret_vault/ -w /usr/share/wordlists/dirb/common.txt
        ```
        
    - Discovered an endpoint: `/administrator`
5. **Accessing the Administrator Panel**
    - Visited `http://<IP>:5000/cupids_secret_vault/administrator`
    - Found a login page.
6. **Credential Guessing**
    - Assumed the username might be `administrator` (common for admin panels).
    - Tried the password hint from `robots.txt`: `cupid_arrow_2026!!!`
    - First attempt with username `administrator` and that password succeeded.
    - Upon login, the flag was displayed.