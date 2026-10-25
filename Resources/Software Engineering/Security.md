# Security

# Some Fundamental Types Of Cryptography

1. **Symmetric Key Cryptography:**
    - In symmetric key cryptography, the same key is used for both encryption and decryption.
    - It's a relatively fast process and is commonly used for securing data transmission.
    - Examples include DES (Data Encryption Standard), AES (Advanced Encryption Standard), and 3DES (Triple DES).
2. **Asymmetric Key Cryptography (Public Key Cryptography):**
    - Asymmetric cryptography involves the use of a pair of keys: a public key for encryption and a private key for decryption.
    - Public keys can be shared openly, while private keys must be kept secret.
    - RSA (Rivest–Shamir–Adleman) and ECC (Elliptic Curve Cryptography) are common asymmetric key algorithms.
3. **Hash Functions:**
    - Hash functions are one-way mathematical functions that transform input data into a fixed-size string of characters.
    - For algorithms like SHA 256, it’s assumed that the chances of getting the same string of 256 bit output for two different input docs is nearly impossible (because 2^256 is such a large number).
    - They are commonly used for data integrity verification and password storage.
    - Examples include SHA-256 (Secure Hash Algorithm) and MD5 (Message Digest Algorithm 5).
    - How they are used for data integrity verification
        - Hash the original message
        - Send the hash along with the message
        - Compute hash at receiver end again using same function, the hash should match
        - This proves message has not been tampered with
4. **Digital Signatures:**
    - Digital signatures are a way to verify the authenticity and integrity of a message or document.
    - They are created using the private key and can be verified using the corresponding public key.
    - Common digital signature algorithms include RSA and DSA (Digital Signature Algorithm).
    - How they are used for authenticity verification
        - The entity that’s generating the digital signature first generates a public-private key pair.
        - A hash function is applied to the message or document to generate a fixed-size hash value.
        - The entity uses its private key to encrypt the hash value of the message. This process is the digital signing, the encrypted hash value is the digital signature.
        - We attach the digital signature with the document.
        - Anyone who receives the signed message can verify its authenticity using the public key of the signer, because he’ll be able to decrypt the signature using public key.
        - Decryption also reveals the original hash value, which can be used to verify integrity of the document.
5. **Public Key Infrastructure (PKI):**
    - PKI is a framework that manages digital keys and certificates.
    - It includes the use of public key cryptography to secure communication and verify the identity of parties involved.
    - Certificate Authorities (CAs) issue digital certificates to verify the association between a public key and an entity.

# Private vs Public Keys Cryptography

| **Private Key** | **Public Key** |
| --- | --- |
| The private key encryption / decryption is faster than the public key. | The public key encryption / decryption is slower than the private key. |
| In this, the same key (secret key) and algorithm are used to encrypt and decrypt the message. | In public-key cryptography, two keys are used, one key is used for encryption, and the other is used for decryption. |
| In private key cryptography, the key is kept a secret. | In public-key cryptography, one of the two keys is kept a secret. |
| The private key is Symmetrical because there is only one key that is called a secret key. | The public key is Asymmetrical because there are two types of keys: private and public keys. |
| In this cryptography, the sender and receiver need to share the same key. | In this cryptography, the sender and receiver do not need to share the same key. |
| In this cryptography, the key is private. | In this cryptography, the public key can be public and a private key is private. |
| It is used for large amounts of text. | It is used for only short messages. |
| There is the possibility of losing the key that renders the systems void. | There is less possibility of key loss, as the key is held publicly. |
| The private key is to be shared between two parties. | The public key can be used by anyone. |
| The Performance testing checks the reliability, scalability, and speed of the system. | The Load testing checks the sustainability of the system. |
| The private key is used in algorithms such as AES 128, AES 192 and AES 256. | The public key is used in algorithms such as RSA, DSA, etc. |
| The private key is kept secret. | The public key is widely distributed. |
| It is used to protect disk drives and other data storage devices. | It is used to secure web sessions and emails. |
| The recipient’s private key decrypts the message. | The recipient’s public key encrypts the message. |
| If the private key is the locking key, then the system can be used to verify documents sent by the holder of the private key. | If the public key is the locking key, then it can be used to send private communication. |