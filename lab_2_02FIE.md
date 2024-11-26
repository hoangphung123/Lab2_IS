
# Lab #2, 20110370, Phung Huy Hoang, INSE331280E_02FIE

# Task 1: Public-Key Based Authentication

This Lab aims to implement public-key based authentication using OpenSSL. The task involves exchanging a challenge message between the client and the server while ensuring authentication using RSA keys.

---

**Question 1**:  
- Implement the public-key based authentication according to the following scheme:
  1. The server generates a challenge message and encrypts it with the client's public key.
  2. The client decrypts the challenge message using its private key.
  3. The client signs the decrypted challenge message with its private key.
  4. The server verifies the signature using the client's public key.

Write step-by-step instructions for each part of the implementation.

---

**Answer 1**:  

## Set up:
- Use vmware to create 2 virtual machines running the ubuntu operating system

<img width="" alt="Screenshot" src="https://github.com/hoangphung123/Lab2_IS/blob/master/setup.png?raw=true"><br>

## Step 1: Generate RSA Keys for Client  

1. **Generate the private key for the client**:  
   Run the following command on the client machine to generate a 2048-bit private key:  
   ```sh
   openssl genrsa -out client_private.pem 2048
   ```  
   - Output: `client_private.pem` (client's private key).

   <img width="" alt="Screenshot" src="https://github.com/hoangphung123/Lab2_IS/blob/master/Privatekey.png?raw=true"><br>

2. **Generate the public key for the client**:  
   Use the private key to derive the public key:  
   ```sh
   openssl rsa -in client_private.pem -pubout -out client_public.pem
   ```  
   - Output: `client_public.pem` (client's public key).

    <img width="" alt="Screenshot" src="https://github.com/hoangphung123/Lab2_IS/blob/master/publicKey.png?raw=true"><br>

## Step 2: Server Creates and Encrypts the Challenge Message  

1. **Create the challenge message**:  
   Create a plaintext file with the challenge message:  
   ```sh
   echo "This is a challenge message. the message is 12021" > challenge.txt
   ```  

   <img width="" alt="Screenshot" src="https://github.com/hoangphung123/Lab2_IS/blob/master/Challenge.png?raw=true"><br>

2. **Encrypt the challenge message with the client's public key**:  
   Encrypt the challenge message to ensure only the client can decrypt it:  
   ```sh
   openssl pkeyutl -encrypt -inkey client_public.pem -pubin -in challenge.txt -out challenge.enc
   ```  
   - Input: `client_public.pem`, `challenge.txt`.  
   - Output: `challenge.enc` (encrypted message).  

   <img width="" alt="Screenshot" src="https://github.com/hoangphung123/Lab2_IS/blob/master/ChallengeEnc.png?raw=true"><br>

3. **Send `challenge.enc` to the client.**

---

## Step 3: Client Decrypts the Challenge Message  

1. **Decrypt the challenge message with the client's private key**:  
   The client uses its private key to decrypt the challenge message:  
   ```sh
   openssl rsautl -decrypt -inkey client_private.pem -in challenge.enc -out decrypted_challenge.txt
   ```  
   - Input: `client_private.pem`, `challenge.enc`.  
   - Output: `decrypted_challenge.txt` (plaintext challenge).  

---

## Step 4: Client Signs the Challenge Message  

1. **Sign the decrypted challenge message**:  
   The client signs the decrypted message to verify its identity:  
   ```sh
   openssl dgst -sha256 -sign client_private.pem -out signed_challenge.sig decrypted_challenge.txt
   ```  
   - Input: `client_private.pem`, `decrypted_challenge.txt`.  
   - Output: `signed_challenge.sig` (signature).  

2. **Send `signed_challenge.sig` to the server.**

---

## Step 5: Server Verifies the Signature  

1. **Verify the signature using the client's public key**:  
   The server verifies the signature to ensure it matches the original message:  
   ```sh
   openssl dgst -sha256 -verify client_public.pem -signature signed_challenge.sig decrypted_challenge.txt
   ```  
   - Input: `client_public.pem`, `signed_challenge.sig`, `decrypted_challenge.txt`.  

   - Output:  
     - If the signature is valid:  
       ```
       Verified OK
       ```  
     - If the signature is invalid:  
       ```
       Verification Failure
       ```

---

**Explanation**:  
- **Why encrypt the challenge message with the client's public key?**  
  To ensure that only the client (with the corresponding private key) can decrypt it.

- **Why sign the decrypted message?**  
  To authenticate the client and prove that the message originated from them.

- **Why verify the signature?**  
  To validate the authenticity of the signed message and ensure it hasn't been tampered with.

