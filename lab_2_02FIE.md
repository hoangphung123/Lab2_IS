# Lab 2, 20110370, Phung Huy Hoang, INSE331280E_02FIE

---

## Task 1: File Transfer with Integrity and Authenticity using OpenSSL

This lab focuses on securely transferring a plaintext file between two machines while ensuring data **integrity** and **authenticity**.

### **Question 1:**

Transfer a plaintext file between two machines and manually ensure its integrity and authenticity using `openssl` at the sender's and receiver's terminals.

### **Answer 1:**

#### **Step 1: Generate Hash for Integrity (Sender)**

1. Create the plaintext file on the sender machine:
   ```bash
   echo "This is a secure file transfer" > file.txt
   ```
2. Create private key and declare key (if not already available):

   ```bash
   openssl genrsa -out private_key.pem 2048
   openssl rsa -in private_key.pem -pubout -out public_key.pem
   ```
3. Create a digital signature for the file:

    ```bash
    openssl dgst -sha256 -sign private_key.pem -out plaintext.sig plaintext.txt
    ```

#### **Step 2: Transfer files and signatures:**

- Send `plaintext.txt` file and `plaintext.sig` signature using SCP

```bash
scp plaintext.txt plaintext.sig user@192.168.x.x:/path/to/destination
```

- At the same time, send the declaration key `public_key.pem` to identify your information.

#### **Step 3: Receiver (Machine B)**

1. Verify signature:

- Make sure to get all three files: `plaintext.txt`, `plaintext.sig`, and `public_key.pem.   ` Verify digital signature:
   ```bash
   openssl dgst -sha256 -verify public_key.pem -signature plaintext.sig plaintext.txt
   ```


## Task 2: File Transfer using Hybrid Encryption (RSA + AES)

This lab demonstrates securely transferring a file using hybrid encryption. The file is encrypted using AES (symmetric encryption), and the AES key is protected using RSA (asymmetric encryption).

---

### **Question 1:**

Transfer a file of your choice between two machines. The file should be encrypted/decrypted using symmetric encryption (AES) with the secret key exchanged using RSA.

### **Answer 1:**

#### **Step 1: Prepare RSA Keys (Receiver)**

1. Generate an RSA private key (2048 bits):
   ```bash
   openssl genrsa -out private_key.pem 2048
   ```
2. Extract the public key from the private key:
   ```bash
   openssl rsa -in private_key.pem -pubout -out public_key.pem
   ```
3. Share the `public_key.pem` file with the sender.

---

#### **Step 2: Encrypt File (Sender)**

1. Create the file to send:

   ```bash
   echo "This is a secret file transfer" > plaintext.txt
   ```

2. Generate a random AES key:

   ```bash
   openssl rand -base64 32 > aes_key.bin
   ```

3. Encrypt the file using AES:

   ```bash
   openssl enc -aes-256-cbc -salt -in plaintext.txt -out ciphertext.enc -pass file:aes_key.bin
   ```

4. Encrypt the AES key using the receiver's RSA public key:

   ```bash
   openssl rsautl -encrypt -inkey public_key.pem -pubin -in aes_key.bin -out encrypted_key.bin
   ```

5. Transfer the encrypted file (`ciphertext.enc`) and encrypted AES key (`encrypted_key.bin`) to the receiver:
   ```bash
   scp ciphertext.enc encrypted_key.bin user@192.168.1.100:/path/to/target/
   ```

---

#### **Step 3: Decrypt File (Receiver)**

1. Decrypt the AES key using the RSA private key:

   ```bash
   openssl rsautl -decrypt -inkey private_key.pem -in encrypted_key.bin -out aes_key_decrypted.bin
   ```

2. Decrypt the file using the decrypted AES key:

   ```bash
   openssl enc -d -aes-256-cbc -in ciphertext.enc -out decrypted_file.txt -pass file:aes_key_decrypted.bin
   ```

3. Verify the content of the decrypted file:
   ```bash
   cat decrypted_file.txt
   ```

---
