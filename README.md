![DHushCP-n](docs/DHushCP-n.png)
# 🛡️ DHushCP-n: DHushCP with Nested Steganography 🛡️

![Python Version](https://img.shields.io/badge/python-3.8%2B-blue.svg)
[![Stable Release](https://img.shields.io/badge/version-0.1.0-blue.svg)](https://github.com/0SINTr/DHushCP-n/releases/tag/v0.1.0)
[![Last Commit](https://img.shields.io/github/last-commit/0SINTr/DHushCP-n)](https://github.com/0SINTr/DHushCP-n/commits/main/)

## 📖 Table of Contents

- [🛡️ DHushCP-n: DHushCP with Nested Steganography](#%EF%B8%8F-dhushcp-covert-communication-via-dhcp-%EF%B8%8F)
  - [🔍 Overview](#-overview)
  - [🚀 DHushCP-n vs. DHushCP](#-dhushcp-n-vs-dhushcp)
  - [🔄 Communication Flow](#-communication-flow)
  - [🖥️ System Requirements](#%EF%B8%8F-system-requirements)
  - [🛠️ Installation and Setup](#%EF%B8%8F-installation--setup)
  - [🎯 Planned Upgrades](#-planned-upgrades)
  - [⚠️ Disclaimer](#%EF%B8%8F-disclaimer)
  - [📜 License](#-license)
  - [📧 Contact](#-professional-collaborations)

## 🔍 Overview

**DHushCP** is a Linux tool designed to facilitate **secure covert wireless communication** between two parties - an Initiator and a Responder - using standard **DHCP (Dynamic Host Configuration Protocol)** packets. 

**DHushCP-n** is a Linux-based, secure communication tool built on top of **DHushCP**. This version introduces **nested text steganography**, enhancing **DHushCP**'s capabilities by embedding covert messages within cover texts using zero-width characters.

Read all about **DHushCP**'s features [**HERE**](https://github.com/0SINTr/DHushCP?tab=readme-ov-file#%EF%B8%8F-dhushcp-covert-communication-via-dhcp-%EF%B8%8F).

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
   git clone https://github.com/0SINTr/DHushCP-n.git
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
**DHushCP-n** is licensed under the [GNU GENERAL PUBLIC LICENSE Version 3](https://github.com/0SINTr/DHushCP-n/blob/main/LICENSE).

## 📧 Professional Collaborations

- **Email Address**:  
  Please direct your inquiries to **sintr.0@pm.me**.

- **Important Guidelines**:  
  - Use a **professional email** or a **ProtonMail** address.
  - Keep your message **concise** and written in **English**.

- **Security Notice**:  
  Emails with **links** or **attachments** will be ignored for security reasons.