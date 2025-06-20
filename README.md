# TLSE — Time-Locked Symmetric Encryption

**Built-in Time. Minute-Based Key. Zero Key Exchange.**

**Author:** Muhammed Shafin P ([@hejhdiss](https://github.com/hejhdiss))  
**License:** Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)  
**Status:** Concept Phase (PoC / Research)

## Overview

TLSE is a time-based symmetric encryption system that generates a unique encryption key every minute, based on the current hour and minute (`HHMM`) combined with a user-defined secret phrase. This phrase is shared out-of-band and never transmitted online. The key auto-expires every 60 seconds. TLSE is designed to operate entirely offline — ideal for secure local or air-gapped environments, CTF systems, and private message exchange. TLSE operates without any key exchange or session handshake over the network. All encryption and decryption processes are handled independently by the binary, using an embedded internal clock reference.

The fundamental innovation of TLSE lies in its elimination of traditional key distribution challenges. Instead of requiring complex cryptographic handshakes or pre-shared keys that must be securely transmitted, TLSE leverages the universal constant of time itself as a synchronization mechanism. Both parties need only agree on a shared secret phrase beforehand, which can be exchanged through any secure channel, and the system automatically generates matching encryption keys based on the current time. This approach significantly reduces the attack surface typically associated with key exchange protocols while maintaining strong cryptographic security through the use of established algorithms like SHA-256 and AES-256.

## Encryption Flow

Both the sender and receiver generate the same symmetric key at any given minute using the formula: `KEY = SHA256(user_secret_phrase + HHMM)`. This key is then used to encrypt or decrypt data using AES-256 (CBC/GCM mode). The binary includes a micro-timer or runtime clock embedded inside itself — it does not depend on the system NTP or network time. Unicode-level obfuscation is applied to the ciphertext output so that even encrypted content avoids detection via common binary patterns or entropy scans. This makes TLSE suitable for stealth use in plain-text transmissions or restricted environments.

The encryption process begins when the TLSE binary reads the current time from its internal clock mechanism, which operates independently of system time to prevent manipulation through NTP spoofing or system clock adjustments. The HHMM value is concatenated with the user's secret phrase and processed through SHA-256 to produce a 256-bit key. This key is then used with AES-256 in either CBC or GCM mode to encrypt the plaintext data. The resulting ciphertext undergoes additional obfuscation to mask its cryptographic nature, making it appear as ordinary text data to automated scanning systems. This multi-layered approach ensures that even if the encrypted data is intercepted, it provides no obvious indicators of its encrypted nature.

## First-Message Sync Logic

If the receiver cannot decrypt a message using the current time-derived key or its ±1 minute fallback, TLSE enters a scan mode. It attempts to decrypt using keys derived from **00:00 to 23:59**, checking each minute until decryption is successful. Once a match is found, that specific minute becomes the receiver's synced reference point for the rest of the session. This means TLSE can auto-sync without any parsing of timestamps or additional metadata — all logic is embedded in encrypted payload handling. This allows TLSE to function as a zero-time-expectation secure communication method, requiring no prior synchronization or pairing step.

The synchronization mechanism is designed to handle real-world scenarios where perfect time alignment between parties may not be achievable. When a receiver encounters an encrypted message that cannot be decrypted with the current minute's key, the system intelligently searches through all possible time-based keys within a 24-hour window. This brute-force approach is computationally feasible because there are only 1,440 possible keys per day (24 hours × 60 minutes). Once successful decryption occurs, the system establishes the sender's time reference and can maintain synchronization for subsequent messages. This eliminates the need for time servers, timestamp headers, or complex synchronization protocols that could compromise the system's stealth characteristics.

## Security Design

Although the system is lightweight and intended for **consumer-level use**, the internal logic is designed with **high-level cryptographic principles**. Each key is ephemeral and session-bound to a single minute. The binary is compiled with full irreversible protections — including **code confusion, internal renaming, hashed function names**, and string obfuscation — to prevent reverse engineering or tampering. The structure and behavior of the binary remain unreadable even under static analysis or disassembly. TLSE can also be paired with external PINs or device-tied secrets for multi-factor validation if desired in future versions.

The security model of TLSE addresses several attack vectors commonly associated with symmetric encryption systems. The minute-based key rotation significantly limits the window of vulnerability for any single key compromise. Even if an attacker successfully extracts a key through cryptanalysis or side-channel attacks, that key becomes useless within 60 seconds. The use of SHA-256 for key derivation provides strong resistance against collision attacks and ensures that even minor changes in the secret phrase result in completely different keys. The binary protection mechanisms prevent attackers from easily analyzing the encryption logic or extracting embedded constants. Forward secrecy is inherently provided since old keys cannot be regenerated without knowledge of the exact time they were used, and the system does not store historical key information.

## Use Cases

TLSE is ideal for local secure messaging, encrypted clipboard sharing, offline keyless device authentication, CTF challenges with time-based unlocking, air-gapped command execution, and secure boot flows in embedded systems. Since no keys are exchanged and each minute has a unique key, there is minimal attack surface and no persistent sessions to exploit. TLSE enables encrypted communication using nothing but shared time and a phrase.

The application domains for TLSE extend beyond traditional encrypted messaging to include scenarios where conventional cryptographic infrastructure is unavailable or undesirable. In air-gapped environments, TLSE provides a means of secure communication without requiring network connectivity for key exchange. For CTF competitions and security training, the time-locked nature creates natural puzzle elements where participants must coordinate timing or deduce temporal relationships. In embedded systems and IoT devices, TLSE offers a lightweight alternative to complex key management systems while maintaining strong security guarantees. The system's independence from external time sources makes it particularly valuable in hostile environments where NTP servers may be compromised or unavailable.

## Status

TLSE is currently in the **conceptual and prototype stage**. Public binaries or demo scripts may be released as part of a PoC or CTF challenge. The project is part of the **BytexGrid Research Series** by Muhammed Shafin P, exploring ultra-secure offline encryption methods without traditional network-based cryptographic dependencies.

Development efforts are focused on creating a robust reference implementation that demonstrates the viability of minute-based symmetric encryption in real-world scenarios. The prototype will include comprehensive testing of the synchronization mechanisms, performance benchmarking of the encryption/decryption processes, and security analysis of the key derivation function. Future research directions include investigating the optimal balance between sync window size and security, exploring integration with hardware security modules for enhanced secret phrase protection, and developing standardized protocols for TLSE adoption in various application domains.

## License

This project is licensed under the **Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)**. You are free to share, adapt, remix, or build upon this project — even commercially — as long as you give proper credit and license your modifications under the same terms.

Full license: https://creativecommons.org/licenses/by-sa/4.0/

The choice of CC BY-SA 4.0 reflects the research-oriented nature of this project and the desire to ensure that improvements and modifications remain available to the broader community. Developers who wish to create commercial implementations are encouraged to consider dual-licensing approaches or to contact the author regarding alternative licensing arrangements. The attribution requirement ensures that the original research contributions are properly recognized while the share-alike provision promotes collaborative development and prevents proprietary capture of community innovations.

## Contributing and Future Development

TLSE represents a bold step toward redefining how secure encryption can work without exchanging keys, requiring synchronization, or relying on internet infrastructure. It offers a promising direction for systems that demand privacy, portability, and operational security. Developers and researchers interested in collaborating, testing, or extending this protocol are welcome to audit and contribute.

The project welcomes contributions in several key areas: cryptographic analysis and security auditing, performance optimization and implementation efficiency, cross-platform compatibility and embedded system support, and formal verification of the synchronization algorithms. Researchers interested in exploring variations of the core concept, such as different time granularities or alternative key derivation functions, are encouraged to fork the project and share their findings. The ultimate goal is to establish TLSE as a viable alternative to traditional key exchange mechanisms in scenarios where simplicity, offline operation, and stealth characteristics are paramount.

**Note:** The author recommends that derivative implementations based on this idea be published under MIT or Apache 2.0 licenses to encourage broader adoption and commercial use.