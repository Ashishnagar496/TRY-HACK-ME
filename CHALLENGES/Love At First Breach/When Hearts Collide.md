# When Hearts Collide

## **Objective**

Exploit a weak file upload mechanism that relies on MD5 hashes for duplicate detection. By generating two different images with the same MD5 hash (hash collision), bypass the duplicate check and retrieve the flag.

## **Methodology**

### **1. Understanding the Vulnerability**

- The application allows image uploads and generates an MD5 hash for each uploaded file.
- If the hash matches an existing image's hash, the application assumes it's a duplicate and triggers a special action (reveals the flag).
- MD5 is cryptographically broken; it is possible to create two different files that produce the same hash using collision attacks.

### **2. Tools Required**

- **fastcoll** – a tool to generate MD5 collisions.
- A base image from the website (or any image) to serve as the original.

### **3. Setting Up fastcoll on Kali Linux**

### **Step 1: Update System**

bash

```
sudo apt update
```

### **Step 2: Install Dependencies**

bash

```
sudo apt install build-essential libboost-all-dev libboost-program-options-dev
```

### **Step 3: Download fastcoll Source**

bash

```
wget https://www.win.tue.nl/hashclash/fastcoll_v1.0.0.5-1_source.zip
```

### **Step 4: Extract Files**

bash

```
unzip fastcoll_v1.0.0.5-1_source.zip
```

*Note: Files extract directly into the current directory.*

### **Step 5: Organize Files (Optional)**

bash

```
mkdir fastcoll_clean
mv block*.cpp main.cpp main.hpp md5.cpp fastcoll_clean/
cd fastcoll_clean
```

### **Step 6: Compile fastcoll**

bash

```
g++ -O3 -DBOOST_TIMER_ENABLE_DEPRECATED *.cpp -lboost_system -lboost_filesystem -lboost_program_options -o fastcoll
```

If compilation succeeds, you will see the executable `fastcoll` in the directory.

### **4. Generating Colliding Images**

### **Step 7: Obtain the Original Image**

- Download the image from the website that is used as the reference (e.g., `dog.jpg`).

bash

```
wget <image_url> -O original.jpg
```

### **Step 8: Generate Two Colliding Images**

Run fastcoll with the original image as a prefix:

bash

```
./fastcoll -p original.jpg -o image1.jpg image2.jpg
```

This creates two files `image1.jpg` and `image2.jpg` that have the same MD5 hash but different content.

### **Step 9: Verify the Collision**

bash

```
md5sum image1.jpg image2.jpg
```

Both outputs should be identical, confirming the collision.

### **5. Exploiting the Application**

### **Step 10: Upload the First Image**

- Upload `image1.jpg` through the application's upload form.
- The server calculates its MD5 hash and stores it.

### **Step 11: Upload the Second Image**

- Upload `image2.jpg`.
- The server calculates its MD5 hash and finds a match with the previously stored hash.
- Because the hashes match, the application treats it as a duplicate and reveals the flag (e.g., in a response message or by granting access).

### **6. Flag Retrieval**

- The flag is displayed on the screen or in the response after the second upload.

## **Flag Obtained**

text

```
THM{...}
```

## **Conclusion**

The application's reliance on MD5 for duplicate detection is insecure due to known collision attacks. By using fastcoll, two different images with identical MD5 hashes were generated, allowing bypass of the duplicate check and retrieval of the flag. This highlights the importance of using cryptographically secure hash functions (e.g., SHA-256) and implementing additional checks (e.g., content comparison) to prevent such attacks.