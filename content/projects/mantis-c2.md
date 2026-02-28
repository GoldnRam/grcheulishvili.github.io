---
title: "Mantis C2 Architecture"
date: 2025-11-27
draft: false
description: "A custom Command and Control framework focusing on signature evasion and resilient communication."
tags: ["MalDev", "Go", "C2", "Polymorphism", "Offensive Research"]
---

## Overview
Mantis C2 is a custom Command and Control framework developed to test and bypass modern EDR telemetry. Commercial and widely available C2 frameworks often carry heavily signatured stubs; Mantis was built from the ground up to minimize static and behavioral footprint.

## 1. Evasion and Persistence Architecture

### AST-Driven Polymorphism Protocol
To defeat static analysis, the builder utilizes a custom source-code mutation protocol:

* The agent is processed by a custom pre-compiler that mutates the AST to generate **structurally unique binaries** for every deployment. This defeats Control Flow Graph (CFG) analysis and static signatures, a key step beyond simple file hash mutation.
* All sensitive strings (e.g., C2 domains, commands) are automatically detected and encrypted at the source-code level and decrypted at runtime using a file-local key.
* Rather than dropping secondary stages to disk, Mantis allocates memory dynamically (using `VirtualAlloc`/`VirtualProtect` equivalents) and executes position-independent shellcode directly.
* All DWARF tables and symbol information are automatically removed during cross-compilation to severely complicate reverse engineering efforts.

### Stealth
* The agent runs silently as a background process.
* Check-in intervals are heavily randomized. During sleep cycles, the agent masks its memory regions to prevent in-memory scanning from flagging dormant malicious threads.

### Self-Contained destruction
The **`quit`** command initiates a self-destruct routine, which is critical for operational cleanup:
1.  Removes Windows Registry Run keys used for persistence.
2.  Securely deletes the executable binary from the disk.
3.  Terminates the session and signals the C2 server.

## 2. Streaming Data Telemetry

The core innovation is the streaming telemetry model, designed for high-volume, low-latency exfiltration:
1. Data is sent immediately in small, encrypted batches over a covert TCP channel. This ensures data capture is continuous, even if the session is unexpectedly terminated.
2. An immediate **Stage 1 Recon Report** is generated upon connection, providing the operator with critical system and file metadata before any manual command is issued.
3. Files exceeding a defined threshold are automatically streamed raw in chunks, bypassing the compression engine to prevent memory spikes and keep CPU load minimal.

## 3. Tooling and Sanitization

The framework includes a custom Builder tool to generate payloads. The code is structured to enforce responsible usage.


> **Disclaimer:** *Core features, including the encryption/decryption keys, C2 IP/Port configuration, and the full multi-threaded session management logic, are omitted from this public documentation to prevent weaponization and maintain responsible disclosure standards. For the technical mechanism behind the polymorphic build, please refer to the dedicated research: [Go AST Polymorphic Engine](https://ieri.sh/research/building-a-polymorphic-engine/).*

-----

*This tool is for educational research and authorized internal threat simulation only.*
