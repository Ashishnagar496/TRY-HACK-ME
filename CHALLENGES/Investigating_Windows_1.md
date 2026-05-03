# TryHackMe Walkthrough: Investigating Windows

**Room Link:** https://tryhackme.com/room/investigatingwindows

---

## Task 1: Initial Access (RDP Connection)

We need to connect to the target Windows machine using **RDP (Remote Desktop Protocol)**.

### Credentials:
- **Username:** Administrator  
- **Password:** letmein123!

---

### For Linux Machine:

Use the following command:bash

xfreerdp /u:Administrator /p:letmein123! /v:<ROOM_IP>

---

### For Windows Machine:

1. Open the **Start Menu** and search for **Remote Desktop Connection**.
2. In the **Computer** field, enter `<ROOM_IP>`.
3. Click **Connect**.
4. If prompted, click **Yes** to proceed.
5. Enter the provided credentials:

   * Username: Administrator
   * Password: letmein123!
6. Make sure you are connected to the **OpenVPN** before attempting this.

---

### Alternative:

You can also use the **TryHackMe AttackBox**, which already has everything configured.

---

## Important Note

`net` is a built-in Windows command-line utility used to manage:

* Users
* Groups
* Network resources
* Services

It is very useful during enumeration.

---

## Question Walkthrough

---

### 1. What is the OS version?

* Open the **Start Menu**
* Search for **System Information**
* Open it and check the OS name

**Answer:**

```
Windows Server 2016
```

---

### 2. What is the last logon time of user `john`?

Run the following command in Command Prompt:

```cmd
net user john
```

* Look for the **Last logon** field

**Answer:**

```
03/02/2019 5:48:32 PM
```

---

### 3. What IP does the system connect to when it first starts?

We check startup programs using the **Registry Editor**.

#### Steps:

1. Open **Run** (`Win + R`)
2. Type:

   ```
   regedit
   ```
3. Navigate to:

   ```
   HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
   ```
4. Look for suspicious entries

* You will find an entry named **updateSvc**
* It contains an IP address

**Answer:**

```
10.32.2.3
```

---

### 4. What two accounts had administrative privileges (excluding Administrator)?

We check users in the **Administrators group**.

```cmd
net localgroup administrators
```

* This lists all users with admin privileges

**Answer:**

```
guest, jenny
```

---

### 5. What is the name of the malicious scheduled task?

We analyze scheduled tasks.

#### Steps:

1. Open **Run**
2. Type:

   ```cmd
   taskschd.msc
   ```
3. Open **Task Scheduler Library**
4. Review all tasks

* Look for suspicious or unusual names

**Answer:**

```
Clean File System
```

---

### 6. What file is used by this malicious task?

1. Double-click the task **Clean File System**
2. Go to the **Actions** tab
3. Observe the executed command

You will see:

```
C:\TMP\nc.ps1 -l 1348
```

* Only the file name is required

**Answer:**

```
nc.ps1
```

---

### 7. What port is being used?

From the same **Actions** tab:

```
-l 1348
```

**Answer:**

```
1348
```

---

### 8. When did Jenny last log on?

```cmd
net user jenny
```

* Check the **Last logon** field

**Answer:**

```
Never
```

---

### 9. At what date did the compromise take place?

* Open **File Explorer**

* Go to **Local Disk (C:)**

* Inspect file timestamps (modified/created dates)

* Multiple suspicious files point to the same date

**Answer:**

```
03/02/2019
```

---

### 10. Which tool was used to dump Windows passwords?

* While using the machine, a CMD window may pop up referencing `/TMP/mim`

#### Steps:

1. Navigate to:

   ```
   C:\TMP
   ```
2. Look for files related to `mim`
3. Open the output file

* The tool name appears in the first line

**Answer:**

```
mimikatz
```

---

### 11. When were special privileges first assigned to a new logon?

We use **Event Viewer**.

#### Steps:

1. Search for **Event Viewer**
2. Go to:

   ```
   Windows Logs → Security
   ```
3. Click **Create Custom View** (right panel)

#### Configure Filter:

* Select **Custom range**
* Set date: **03/02/2019**
* Set time: **4:00 PM – 4:30 PM**

4. Under **Event Logs**, select:

   ```
   Windows Logs → Security
   ```
5. Apply and save the filter

#### Analysis:

* Look for events under **Task Category: Security Group Management**
* Identify when privileges were assigned

**Answer:**

```
(Find exact timestamp in logs)
```

---

### 12. What was the attacker's external C2 server IP?

Check the **hosts file**.

#### Steps:

1. Navigate to:

   ```
   C:\Windows\System32\drivers\etc
   ```
2. Open the **hosts** file using Notepad
3. Look for suspicious entries

* You will see an entry for:

  ```
  google.com
  ```

#### Verification:

Run:

```cmd
ping google.com
```

* Compare the IP from ping vs hosts file
* If different → hosts file IP is malicious

**Answer:**

```
(IP found in hosts file)
```

---

### 13. What was the extension of the uploaded shell?

#### Steps:

1. Navigate to:

   ```
   C:\inetpub\wwwroot
   ```
2. Inspect files

* You will see:

  * `.jsp` files
  * `.gif` file

* `.jsp` files can execute server-side code

**Answer:**

```
.jsp
```

---

### 14. What was the last port the attacker opened?

Check **Windows Firewall rules**.

#### Steps:

1. Open **Windows Defender Firewall**

2. Go to **Inbound Rules**

3. Filter:

   * By **Group**
   * Select **Rules without group**

4. Look for suspicious rule:

   ```
   Allow outside connection for development
   ```

5. Open it → go to **Protocols and Ports**

* Check **Local Port**

**Answer:**

```
1337
```

---

### 15. Which website was used for DNS poisoning?

* Found in the **hosts file**

* The attacker mapped a domain manually

**Answer:**

```
google.com
```

---

## Summary

This room covers:

* RDP access to Windows systems
* User and group enumeration using `net`
* Registry persistence mechanisms
* Scheduled task analysis
* Event log investigation
* Detecting credential dumping (Mimikatz)
* DNS poisoning via hosts file
* Firewall rule analysis

---

```
```
