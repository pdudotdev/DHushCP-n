![DHushCP-n](docs/DHushCP-n.png)
# 🛡️ DHushCP-n: DHushCP with Nested Steganography 🛡️

![Python Version](https://img.shields.io/badge/python-3.8%2B-blue.svg)
[![Stable Release](https://img.shields.io/badge/version-0.1.0-blue.svg)](https://github.com/pdudotdev/DHushCP-n/releases/tag/v0.1.0)
[![Last Commit](https://img.shields.io/github/last-commit/pdudotdev/DHushCP-n)](https://github.com/pdudotdev/DHushCP-n/commits/main/)

## 📖 Table of Contents

- [🛡️ DHushCP-n: DHushCP with Nested Steganography](#%EF%B8%8F-dhushcp-covert-communication-via-dhcp-%EF%B8%8F)
  - [🔍 Overview](#-overview)
  - [🚀 DHushCP-n vs. DHushCP](#-dhushcp-n-vs-dhushcp)
  - [🔄 Communication Flow](#-communication-flow)
  - [🔐 Encryption and Hashing Algorithms](#-encryption-and-hashing-algorithms)
  - [🖥️ System Requirements](#%EF%B8%8F-system-requirements)
  - [🛠️ Installation and Setup](#%EF%B8%8F-installation--setup)
  - [🎯 Planned Upgrades](#-planned-upgrades)
  - [⚠️ Disclaimer](#%EF%B8%8F-disclaimer)
  - [📜 License](#-license)

## 🔍 Overview

**DHushCP** is a Linux tool designed to facilitate **secure covert wireless communication** between two parties - an Initiator and a Responder - using standard **DHCP (Dynamic Host Configuration Protocol)** packets. 

**DHushCP-n** is a Linux-based, secure communication tool built on top of **DHushCP**. This version introduces **nested text steganography**, enhancing **DHushCP**'s capabilities by embedding covert messages within cover texts using zero-width characters.

Read all about **DHushCP**'s features [**HERE**](https://github.com/pdudotdev/DHushCP?tab=readme-ov-file#%EF%B8%8F-dhushcp-covert-communication-via-dhcp-%EF%B8%8F).

🍀 **NOTE:** This is an ongoing **research project** for educational purposes rather than a full-fledged production-ready tool, so treat it accordingly.

## 🚀 DHushCP-n vs. DHushCP

The original **DHushCP** secures the key and message exchange between the **Initiator** and the **Responder** by embedding the encrypted message in Option 226 of DHCP Discover packets.

### Benefits

With the original **DHushCP**, following the ECC-based key exchange the **plaintext message** entered by the user is:
- Encrypted using the shared **AES** key with **AES-GCM** with a **SHA-256** checksum appended
- Obfuscated using **Network Steganography** principles inside **DHCP Option 226**

***NEW!*** Now **DHushCP-n** adds yet another layer of obfuscation by using **Text Steganography**.<br /><br />
Therefore, with **DHushCP-n** the **plaintext message** entered by the user is:
- Embeded into a predefined 8-character `cover_text` string using zero-width characters
- Encrypted using the shared **AES** key with **AES-GCM** with a **SHA-256** checksum appended
- Obfuscated using **Network Steganography** principles inside **DHCP Option 226**

### Limitations 

Unlike the original **DHushCP**, the **DHushCP-n** version comes with some limitations:
- **Limited Per-Packet Capacity:** With an 8-character cover text, only 15 characters (secret message) can be embedded per DHCP Discover packet. Transmitting longer messages requires sending multiple packets, which may be noticeable under certain network conditions.
- **Potential Stripping by Systems:** Some systems, applications, or network devices might inadvertently strip or alter zero-width characters, disrupting message integrity or making embedded messages inaccessible.

## 🔄 Communication Flow

1. **Initial Exchange:**
   - **Initiator:**
     - Generates a unique session ID.
     - Detects and selects the active wireless interface.
     - Generates an ECC key pair (private/public keys).
     - Embeds its public key, the DHUSHCP_ID (option 224), and session ID (option 225) into the DHCP Discover packet.
     - Sends the DHCP Discover packet and waits for the Responder's public key.
   
   - **Responder:**
     - Listens for DHCP Discover packets with option 224 set to DHUSHCP_ID.
     - Upon receiving a valid DHCP Discover (option 224 set to DHUSHCP_ID), extracts the session ID from option 225.
     - Extracts and reassembles the Initiator's public ECC key from the correct DHCP option.
     - Generates its own ECC key pair.
     - Embeds its public key, the DHUSHCP_ID, and the extracted session ID into a new DHCP Discover packet.
     - Sends the DHCP Discover and waits for Initiator's message.

2. **Message Transmission:**
   - **Initiator:**
     - Receives the Responder's public key from the DHCP Discover packet.
     - Derives the shared AES key using its private ECC key and the Responder's public ECC key.
     - Prompts the user to input a message to send to the Responder.
     - **Embeds the secret message into an 8-character cover_text using zero-width characters.**
     - Encrypts the **stego message** using the shared AES key with AES-GCM and adds a SHA-256 checksum.
     - Embeds the encrypted message with the checksum and session ID into a new DHCP Discover packet.
     - Sends the DHCP Discover packet containing the encrypted message.
   
   - **Responder:**
     - Receives the encrypted DHCP Discover packet from the Initiator.
     - Extracts and decrypts the message using the shared AES key.
     - **Decodes the stego message to retrieve the original secret message.**
     - Displays the decrypted message to the Responder user.
     - Prompts the Responder user to input a reply message.
     - **Embeds the secret message into an 8-character cover_text using zero-width characters.**
     - Encrypts the reply using the shared AES key with AES-GCM and appends a SHA-256 checksum.
     - Embeds the encrypted reply with the checksum and session ID into a new DHCP Discover packet.
     - Sends the DHCP Discover packet containing the encrypted reply.

3. **Finalization:**
   - **Initiator:**
     - Receives the encrypted DHCP Discover packet containing the Responder's reply.
     - Decrypts the reply using the shared AES key.
     - **Decodes the stego message to retrieve the original secret message.**
     - Displays the decrypted reply message to the Initiator user.
     - Upon request (`Ctrl+C`), performs cleanup by deleting encryption keys, clearing system logs (syslog, auth), and resetting the terminal.
   
   - **Responder:**
     - Upon request (`Ctrl+C`), performs cleanup by deleting encryption keys, clearing system logs (syslog, auth), and resetting the terminal.

## 🔐 **Encryption and Hashing Algorithms**

### Reasons for Choosing the Encryption and Hashing Algorithms

DHushCP utilizes a combination of advanced cryptographic algorithms to ensure secure and covert communication over local networks. The chosen algorithms are selected based on their security strength, efficiency, and suitability for embedding within DHCP packets.

#### 1. Elliptic Curve Cryptography (ECC)

- **Efficiency**: ECC offers equivalent security to traditional algorithms like RSA but with smaller key sizes. This results in faster computations and reduced overhead, which is essential when embedding data within the limited space of DHCP packets.
- **Security**: The `SECP384R1` elliptic curve used in DHushCP provides robust security against modern cryptographic attacks.
- **Key Exchange**: ECC enables secure key exchange through the Elliptic Curve Diffie-Hellman (ECDH) algorithm, allowing both parties to establish a shared secret without transmitting the secret itself over the network.

#### 2. Advanced Encryption Standard (AES) in Galois/Counter Mode (GCM)

- **Confidentiality and Integrity**: AES-GCM provides both encryption (confidentiality) and authentication (integrity) in a single, efficient step. It ensures that the message remains confidential and detects any tampering.
- **Performance**: AES is widely adopted and optimized in hardware and software, offering excellent performance for encrypting data.
- **Security**: GCM mode is resistant to various cryptographic attacks and, when used correctly with a unique nonce, provides strong security guarantees.

#### 3. HMAC-based Extract-and-Expand Key Derivation Function (HKDF)

- **Key Derivation**: HKDF is used to derive a strong symmetric encryption key from the shared secret established via ECDH. It ensures the derived key is suitable for encryption purposes.
- **Standardization**: HKDF is a standardized method for key derivation in cryptographic protocols, providing interoperability and security.

#### 4. Secure Hash Algorithm 256 (SHA-256)

- **Integrity Verification**: SHA-256 generates a cryptographic hash of the message, which is used to verify that the message has not been altered during transmission.
- **Collision Resistance**: It is computationally infeasible to find two different messages that produce the same hash, ensuring the uniqueness and integrity of the message.

### How Each Encryption/Hashing Operation Secures Communication

#### 1. Key Exchange and Shared Secret Derivation

- **ECC Key Pair Generation**: Both the Initiator and Responder generate their own ECC key pairs (private and public keys) using the `SECP384R1` curve.

  ```python
  private_key = ec.generate_private_key(ec.SECP384R1())
  public_key = private_key.public_key()
  ```

- **Public Key Exchange**: The parties exchange their public keys embedded within DHCP Discover packets using custom DHCP options.

- **Shared Secret Derivation**: Each party uses their private key and the other's public key to compute a shared secret via the ECDH algorithm.

  ```python
  shared_secret = private_key.exchange(ec.ECDH(), peer_public_key)
  ```

- **AES Key Derivation with HKDF**: The shared secret is passed through HKDF to derive a symmetric AES-256 key for encrypting and decrypting messages.

  ```python
  derived_key = HKDF(
      algorithm=hashes.SHA256(),
      length=32,
      salt=None,
      info=b'DHushCP-SharedKey',
  ).derive(shared_secret)
  ```

#### 2. Message Encryption Process

- **Plaintext Preparation**: The sender prepares the plaintext message to be sent.

- **Checksum Calculation with SHA-256**: A SHA-256 checksum (hash) of the plaintext is computed and appended to the message. This checksum will be used by the recipient to verify the integrity of the message.

  ```python
  digest = hashes.Hash(hashes.SHA256())
  digest.update(plaintext.encode())
  checksum = digest.finalize()
  ```

- **Nonce Generation**: A unique 12-byte nonce is generated for use with AES-GCM encryption. The nonce ensures that each encryption operation is unique, which is critical for the security of GCM mode.

  ```python
  nonce = os.urandom(12)
  ```

- **AES-GCM Encryption**: The plaintext message is encrypted using AES-256 in GCM mode with the derived symmetric key and the generated nonce.

  ```python
  aesgcm = AESGCM(aes_key)
  ciphertext = aesgcm.encrypt(nonce, plaintext.encode(), None)
  ```

- **Encrypted Package Creation**: The nonce, ciphertext, and checksum are combined into a single package ready for transmission.

  ```python
  encrypted_package = nonce + ciphertext + checksum
  ```

#### 3. Message Transmission

- **Embedding in DHCP Packets**: The encrypted package is embedded into DHCP Discover packets using a custom DHCP option (option 226). This allows the message to be transmitted over the network covertly, blending in with regular DHCP traffic.

  ```python
  options = [(DATA_OPTION, encrypted_package)]
  packet = create_dhcp_discover(session_id, dhushcp_id, options)
  send_dhcp_discover(packet, iface)
  ```

#### 4. Message Decryption Process

- **Packet Reception**: The recipient detects the DHCP Discover packet containing the encrypted message.

- **Extraction of Encrypted Package**: The encrypted package is extracted from the DHCP options.

- **Nonce and Ciphertext Separation**: The nonce and ciphertext are separated from the received package.

  ```python
  nonce = encrypted_package[:12]
  ciphertext = encrypted_package[12:-32]
  received_checksum = encrypted_package[-32:]
  ```

- **AES-GCM Decryption**: Using the shared symmetric key and the nonce, the recipient decrypts the ciphertext to retrieve the plaintext message.

  ```python
  plaintext_bytes = aesgcm.decrypt(nonce, ciphertext, None)
  plaintext = plaintext_bytes.decode()
  ```

- **Checksum Verification**: The recipient computes the SHA-256 checksum of the decrypted plaintext and compares it to the received checksum to verify that the message has not been altered.

  ```python
  digest = hashes.Hash(hashes.SHA256())
  digest.update(plaintext.encode())
  calculated_checksum = digest.finalize()

  if calculated_checksum != received_checksum:
      print("Checksum verification failed!")
      return None
  ```

- **Plaintext Retrieval**: If the checksum matches, the recipient accepts the message as authentic and unaltered.

#### 5. Security Benefits

- **Confidentiality**: AES-256 encryption ensures that only parties with the correct symmetric key can decrypt the messages, keeping the content confidential.

- **Integrity**: The use of AES-GCM's authentication and an additional SHA-256 checksum ensures that any tampering with the message during transmission is detectable.

- **Authentication**: Since the shared key is derived from the ECDH key exchange using private keys, only the intended parties can compute the shared secret, authenticating each other.

- **Replay Protection**: The use of unique nonces for each message prevents replay attacks, as the same nonce cannot be reused without detection.

- **Stealth**: Embedding encrypted messages within DHCP Discover packets makes the communication less noticeable to network monitoring systems, as it appears as regular network traffic.

By combining these encryption and hashing algorithms, DHushCP provides a secure communication channel that is both confidential and resilient against common network attacks, all while maintaining a low profile within standard network operations.

## 🖥️ System Requirements

- **Operating System:** Linux-based systems (e.g., Ubuntu, Debian, Kali)
  - Latest release thoroughly tested and functional on **Ubuntu 24.04**.
- **Python Version:** Python 3.8 or higher
- **Dependencies:**
  - `scapy` for packet crafting and sniffing
  - `cryptography` for ECC encryption and checksum generation
- **Privileges:** Root or sudo access to send and receive DHCP packets
- **Network Interface:** Active wireless interface in UP state

## 🛠️ Installation & Setup

1. **Clone the Repository:** Use the commands below in your Linux terminal.
   ```bash
   git clone https://github.com/pdudotdev/DHushCP-n.git
   cd DHushCP
   ```

2. **Install Dependencies:** Ensure you have Python 3.8 or higher installed. Then, install the required Python packages.
   ```bash
   sudo apt install python3-scapy
   sudo apt install python3-cryptography
   ```

3. **Configure Wireless Interface:** Ensure that your wireless interface is active and in the UP state. **DHushCP** will automatically detect and prompt you to select the active interface if multiple are detected.

4. **Run the Scripts:** Both Initiator and Responder scripts require root privileges to send and sniff DHCP packets. You can run the scripts using `sudo`:

**Responder:**
```
   set +o history
   sudo python3 responder.py --id DHUSHCP_ID
```

**Initiator:**
```
   set +o history
   sudo python3 initiator.py --id DHUSHCP_ID
```

Follow the on-screen prompts on the **Initiator** to initiate and manage the communication session. Make sure the **Responder** is already listening.

## 🎯 Planned Upgrades
- [x] Improved CLI experience
- [ ] DER encoding vs. PEM now
- [ ] More testing is needed

## ⚠️ Disclaimer
**DHushCP-n** is intended for educational and authorized security testing purposes only. Unauthorized interception or manipulation of network traffic is illegal and unethical. Users are responsible for ensuring that their use of this tool complies with all applicable laws and regulations. The developers of **DHushCP-n** do not endorse or support any malicious or unauthorized activities. Use this tool responsibly and at your own risk.

## 📜 License
**DHushCP-n** is licensed under the [GNU GENERAL PUBLIC LICENSE Version 3](https://github.com/pdudotdev/DHushCP-n/blob/main/LICENSE).