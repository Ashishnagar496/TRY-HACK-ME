# Signed Messages

## **Objective**

Exploit a flaw in the deterministic RSA key generation of a secure messaging application to forge a valid signature for the admin user and retrieve the flag.

## **Application Overview**

- The website "LoveNote" allows users to send encrypted love messages using RSA-2048 digital signatures.
- Upon registration, each user receives a unique RSA keypair. Messages are signed using the private key, and recipients verify using the public key.
- The "Learn More" page reveals the technology stack: Python, Flask, cryptography library, RSA-2048, SHA-256, and PSS padding.
- Additional functionality: a "Compose" page to create signed messages, and a "Verify" page to check signatures.

## **Reconnaissance**

- Registered a test account and noted the generated public/private keys.
- Intercepted all requests using Burp Suite to understand the application's behavior.
- Performed directory enumeration with `dirsearch` and discovered a `/debug` endpoint.
- The `/debug` page displayed system debug logs, which provided insight into the key generation process.

## **Vulnerability Discovery**

From the debug logs and the application's description, it became clear that the RSA key generation is **deterministic**. The private key is derived solely from the username using a fixed seed pattern and prime derivation. This means that anyone can recompute the private key for any user if they know the algorithm.

The key generation logic (as inferred from the source or debug logs) is as follows:

- Seed pattern: `"{username}_lovenote_2026_valentine"`
- Public exponent `E = 65537`
- Primes `p` and `q` are derived by taking the SHA-256 hash of the seed (and seed + "pki") and then finding the next prime.
- The private exponent `d` is computed using the Carmichael function.

Because the process is deterministic, the private key for the **admin** user can be reconstructed.

## **Exploitation**

### **Reconstructing the Admin Private Key**

A Python script was written to replicate the key generation algorithm and produce a signature for any message using the admin's private key.

python

```
import hashlib
from sympy import nextprime
from math import gcd
from Crypto.PublicKey import RSA

# Cryptography Library Imports
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives import serialization

# --- PART 1: Key Generation (Exact Logic from the Application) ---
SEED_PATTERN = "{username}_lovenote_2026_valentine"
E = 65537

def sha256_int(data: bytes) -> int:
    return int.from_bytes(hashlib.sha256(data).digest(), "big")

def derive_prime(seed: bytes) -> int:
    return int(nextprime(sha256_int(seed)))

def lcm(a: int, b: int) -> int:
    return a * b // gcd(a, b)

def generate_keypair_object(username: str) -> RSA.RsaKey:
    # deterministic seed
    seed = SEED_PATTERN.format(username=username).encode()

    # derive primes
    p = derive_prime(seed)
    q = derive_prime(seed + b"pki")

    if p == q:
        q = nextprime(q + 1)

    n = p * q
    lam = lcm(p - 1, q - 1)
    d = pow(E, -1, lam)

    # Return the PyCryptodome Key Object
    return RSA.construct((n, E, d, p, q))

# --- PART 2: Main Execution & Signing ---
if __name__ == "__main__":
    # 1. Get Inputs
    username = input("Username: ").strip()
    message = input("Message: ").strip()

    # 2. Generate the Key (PyCryptodome)
    print(f"\n[*] Generating deterministic key for '{username}'...")
    key_obj = generate_keypair_object(username)

    # 3. Bridge: Export to PEM so 'cryptography' can read it
    pem_data = key_obj.export_key(pkcs=8)

    # 4. Load into 'cryptography' library
    private_key = serialization.load_pem_private_key(
        pem_data,
        password=None
    )

    # 5. Sign using PSS with MAX_LENGTH salt (as per application)
    print(f"[*] Signing message using PSS (MAX_LENGTH)...")
    try:
        signature = private_key.sign(
            message.encode('utf-8'),
            padding.PSS(
                mgf=padding.MGF1(hashes.SHA256()),
                salt_length=padding.PSS.MAX_LENGTH
            ),
            hashes.SHA256()
        )

        print("\n" + "="*50)
        print("SUCCESS! Use these details on the Verify page:")
        print(f"Sender:{username}")
        print(f"Message:{message}")
        print(f"Signature:{signature.hex()}")
        print("="*50)

    except Exception as e:
        print(f"Error during signing:{e}")
```

### **Obtaining the Flag**

- Ran the script and entered `admin` as the username. Any message can be used (e.g., "test").
- The script output a hexadecimal signature.
- Navigated to the application's "Verify" page.
- Selected sender `admin`, entered any random message (the same one used in the script), and pasted the generated signature.
- Upon verification, the application accepted the forged signature and displayed the flag.

## **Flag Obtained**

text

```
THM{...}
```

## **Conclusion**

The vulnerability stemmed from **deterministic RSA key generation** based solely on the username. This allowed an attacker to reconstruct any user's private key, including that of the administrator, and forge valid signatures. The flaw could have been avoided by using cryptographically secure random number generation for key pairs, ensuring that private keys are not predictable.