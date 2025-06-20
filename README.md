# TLSE — Time-Locked Symmetric Encryption

**Built-in Time. Minute-Based Key. Zero Key Exchange.**

**Author:** Muhammed Shafin P ([@hejhdiss](https://github.com/hejhdiss))  
**License:** Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)  
**Status:** Concept Phase (PoC / Research)

## Overview

TLSE is a time-based symmetric encryption system that generates a unique encryption key every minute, based on the current hour and minute (`HHMM`) combined with a user-defined secret phrase. This phrase is shared out-of-band and never transmitted online. The key auto-expires every 60 seconds. TLSE is designed to operate entirely offline — ideal for secure local or air-gapped environments, CTF systems, and private message exchange. TLSE operates without any key exchange or session handshake over the network. All encryption and decryption processes are handled independently by the binary, using an embedded internal clock reference.

The fundamental innovation of TLSE lies in its elimination of traditional key distribution challenges. Instead of requiring complex cryptographic handshakes or pre-shared keys that must be securely transmitted, TLSE leverages the universal constant of time itself as a synchronization mechanism. Both parties need only agree on a shared secret phrase beforehand, which can be exchanged through any secure channel, and the system automatically generates matching encryption keys based on the current time. This approach significantly reduces the attack surface typically associated with key exchange protocols while maintaining strong cryptographic security through the use of established algorithms like SHA-256 and AES-256.

## Encryption Flow

Both the sender and receiver generate the same symmetric key at any given minute using a proprietary formula that extends beyond the basic `KEY = SHA256(user_secret_phrase + HHMM)`. **The actual key generation process includes a sophisticated encoding phase that applies creator-specific transformations known only to the binary itself.** This encoding layer adds an additional security dimension that remains completely opaque to external analysis.

The complete key derivation process follows this enhanced flow:
1. **Base Concatenation**: `user_secret_phrase + HHMM`
2. **Proprietary Encoding**: Application of creator-specific transformation algorithms
3. **Hash Generation**: SHA-256 processing of the encoded string
4. **Final Key**: 256-bit encryption key for AES-256 operations

This key is then used to encrypt or decrypt data using AES-256 (CBC/GCM mode). The binary includes a micro-timer or runtime clock embedded inside itself — it does not depend on the system NTP or network time. Unicode-level obfuscation is applied to the ciphertext output so that even encrypted content avoids detection via common binary patterns or entropy scans. This makes TLSE suitable for stealth use in plain-text transmissions or restricted environments.

The encryption process begins when the TLSE binary reads the current time from its internal clock mechanism, which operates independently of system time to prevent manipulation through NTP spoofing or system clock adjustments. The HHMM value is concatenated with the user's secret phrase and processed through the proprietary encoding phase before SHA-256 hashing to produce the final 256-bit key. This key is then used with AES-256 in either CBC or GCM mode to encrypt the plaintext data. The resulting ciphertext undergoes additional obfuscation to mask its cryptographic nature, making it appear as ordinary text data to automated scanning systems. This multi-layered approach ensures that even if the encrypted data is intercepted, it provides no obvious indicators of its encrypted nature.

## First-Message Sync Logic

If the receiver cannot decrypt a message using the current time-derived key or its ±1 minute fallback, TLSE enters a scan mode. It attempts to decrypt using keys derived from **00:00 to 23:59**, checking each minute until decryption is successful. Once a match is found, that specific minute becomes the receiver's synced reference point for the rest of the session. This means TLSE can auto-sync without any parsing of timestamps or additional metadata — all logic is embedded in encrypted payload handling. This allows TLSE to function as a zero-time-expectation secure communication method, requiring no prior synchronization or pairing step.

The synchronization mechanism is designed to handle real-world scenarios where perfect time alignment between parties may not be achievable. When a receiver encounters an encrypted message that cannot be decrypted with the current minute's key, the system intelligently searches through all possible time-based keys within a 24-hour window. This brute-force approach is computationally feasible because there are only 1,440 possible keys per day (24 hours × 60 minutes). Once successful decryption occurs, the system establishes the sender's time reference and can maintain synchronization for subsequent messages. This eliminates the need for time servers, timestamp headers, or complex synchronization protocols that could compromise the system's stealth characteristics.

## Advanced Security Design

Although the system is lightweight and intended for **consumer-level use**, the internal logic is designed with **military-grade cryptographic principles**. Each key is ephemeral and session-bound to a single minute. 

### Binary Protection Mechanisms

The binary is compiled with comprehensive irreversible protections that exceed standard obfuscation techniques:

- **Complete Code Confusion**: All function logic is scrambled using advanced control flow obfuscation
- **Internal Symbol Renaming**: Every function, variable, and constant is renamed with misleading identifiers
- **Hash-Based Function Names**: Function names are replaced with cryptographic hashes to prevent semantic analysis  
- **String Obfuscation**: All embedded strings are encrypted and decoded at runtime
- **Misleading Binary Contents**: Fake functions, dummy variables, and red herring code paths are injected throughout
- **Anti-Analysis Measures**: Detection and countermeasures against debugging, disassembly, and reverse engineering tools
- **Proprietary Encoding Concealment**: The creator-specific encoding algorithms are buried within layers of obfuscated code

### Enhanced Key Security

The **proprietary encoding phase** in key generation represents the most critical security enhancement. This encoding process:

- Applies transformation algorithms known exclusively to the binary creator
- Cannot be reverse-engineered even through complete binary disassembly
- Renders the key generation process completely unique to TLSE binaries
- Ensures that even if the basic `SHA256(phrase + HHMM)` formula is discovered, unauthorized implementations cannot generate valid keys
- Provides an additional layer of authentication, as only genuine TLSE binaries can produce compatible keys

The structure and behavior of the binary remain unreadable even under sophisticated static analysis or dynamic disassembly. TLSE can also be paired with external PINs or device-tied secrets for multi-factor validation if desired in future versions.

The security model of TLSE addresses several attack vectors commonly associated with symmetric encryption systems. The minute-based key rotation significantly limits the window of vulnerability for any single key compromise. Even if an attacker successfully extracts a key through cryptanalysis or side-channel attacks, that key becomes useless within 60 seconds. The use of SHA-256 for key derivation provides strong resistance against collision attacks and ensures that even minor changes in the secret phrase result in completely different keys. The proprietary encoding phase adds an insurmountable barrier to unauthorized key generation. The comprehensive binary protection mechanisms prevent attackers from analyzing the encryption logic, extracting embedded constants, or understanding the proprietary encoding algorithms. Forward secrecy is inherently provided since old keys cannot be regenerated without knowledge of the exact time they were used and access to the proprietary encoding process.

## Use Cases

TLSE is ideal for local secure messaging, encrypted clipboard sharing, offline keyless device authentication, CTF challenges with time-based unlocking, air-gapped command execution, and secure boot flows in embedded systems. Since no keys are exchanged and each minute has a unique key generated through proprietary encoding, there is minimal attack surface and no persistent sessions to exploit. TLSE enables encrypted communication using nothing but shared time, a phrase, and the creator's proprietary encoding process embedded in the binary.

The application domains for TLSE extend beyond traditional encrypted messaging to include scenarios where conventional cryptographic infrastructure is unavailable or undesirable. In air-gapped environments, TLSE provides a means of secure communication without requiring network connectivity for key exchange. For CTF competitions and security training, the time-locked nature creates natural puzzle elements where participants must coordinate timing or deduce temporal relationships. In embedded systems and IoT devices, TLSE offers a lightweight alternative to complex key management systems while maintaining strong security guarantees. The system's independence from external time sources and reliance on proprietary encoding makes it particularly valuable in hostile environments where NTP servers may be compromised or unauthorized implementations might be attempted.

## Status

TLSE is currently in the **conceptual and prototype stage**. Public binaries or demo scripts may be released as part of a PoC or CTF challenge. The project is part of the **Amptemp Research Series** by Muhammed Shafin P, exploring ultra-secure offline encryption methods without traditional network-based cryptographic dependencies.

Development efforts are focused on creating a robust reference implementation that demonstrates the viability of minute-based symmetric encryption with proprietary encoding in real-world scenarios. The prototype will include comprehensive testing of the synchronization mechanisms, performance benchmarking of the encryption/decryption processes, and security analysis of the enhanced key derivation function. Future research directions include investigating the optimal balance between sync window size and security, exploring integration with hardware security modules for enhanced secret phrase protection, and developing standardized protocols for TLSE adoption in various application domains while maintaining the proprietary encoding advantages.

## License

This project is licensed under the **Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)**. You are free to share, adapt, remix, or build upon this project — even commercially — as long as you give proper credit and license your modifications under the same terms.

**Important Note**: The proprietary encoding algorithms and binary obfuscation techniques described in this documentation are implementation-specific and remain the intellectual property of the creator. While the conceptual framework is open-source, the actual encoding methods are not disclosed and cannot be replicated without access to the original implementation.

Full license: https://creativecommons.org/licenses/by-sa/4.0/

The choice of CC BY-SA 4.0 reflects the research-oriented nature of this project and the desire to ensure that improvements and modifications remain available to the broader community. Developers who wish to create commercial implementations are encouraged to consider dual-licensing approaches or to contact the author regarding alternative licensing arrangements. The attribution requirement ensures that the original research contributions are properly recognized while the share-alike provision promotes collaborative development and prevents proprietary capture of community innovations.

## Contributing and Future Development

TLSE represents a bold step toward redefining how secure encryption can work without exchanging keys, requiring synchronization, or relying on internet infrastructure. It offers a promising direction for systems that demand privacy, portability, and operational security. The proprietary encoding enhancement ensures that TLSE implementations remain secure against unauthorized replication while maintaining the open-source philosophy for research and development.

Developers and researchers interested in collaborating, testing, or extending this protocol are welcome to audit and contribute to the conceptual framework. However, access to the proprietary encoding specifications requires direct collaboration with the creator.

The project welcomes contributions in several key areas: cryptographic analysis and security auditing of the public components, performance optimization and implementation efficiency, cross-platform compatibility and embedded system support, and formal verification of the synchronization algorithms. Researchers interested in exploring variations of the core concept, such as different time granularities or alternative key derivation functions, are encouraged to fork the project and share their findings. The ultimate goal is to establish TLSE as a viable alternative to traditional key exchange mechanisms in scenarios where simplicity, offline operation, and stealth characteristics are paramount.

**Note:** The author recommends that derivative implementations based on this idea be published under MIT or Apache 2.0 licenses to encourage broader adoption and commercial use, while respecting the proprietary nature of the encoding algorithms.