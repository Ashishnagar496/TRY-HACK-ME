# Love Letter Locker

## **Objective**

Exploit an Insecure Direct Object Reference (IDOR) vulnerability to access other users' private letters and retrieve the flag.

## **Methodology**

### **1. Account Creation and Exploration**

- Registered a new user account on the application.
- Intercepted and analyzed all requests using Burp Suite to understand the application's functionality.
- Logged in and navigated to the "My Letters" section.

### **2. Creating a Letter**

- Created a new letter and saved it.
- Clicked the "Open Letter" button for the newly created letter.
- Observed the following GET request:
    
    text
    
    ```
    GET /letter/3 HTTP/1.1
    ```
    
    The numeric value in the URL path represents the letter ID.
    

### **3. IDOR Exploitation**

- Modified the letter ID in the request from `3` to `2` and sent the request.
- Received another letter's content, confirming that the application does not properly enforce ownership checks.
- Changed the ID to `1` and sent the request.
- The response contained a letter with the flag.

### **4. Flag Retrieval**

- Searched for the string `THM` in the response body.
- Located and copied the flag.

## **Flag Obtained**

text

```
THM{...}
```

## **Conclusion**

The application suffered from an Insecure Direct Object Reference vulnerability, allowing any authenticated user to access letters belonging to other users by simply changing the numeric identifier in the URL. This highlights the need for proper access control checks on all object-level operations.