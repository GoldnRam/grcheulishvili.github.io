---
title: "Project: High-Fidelity Stealth Agent - Research in Evasion and C2 Telemetry"
date: 2025-11-27
draft: false
description: "An R&D project detailing a self-destructing, polymorphic Go-based agent designed to stress-test EDR and Blue Team detection strategies."
tags: ["MalDev", "Go", "C2", "Polymorphism", "Offensive Research"]
---

## Overview
This project is a Command & Control (C2) framework built as a research vehicle for testing modern evasion techniques and high-fidelity data extraction methods. The agent is designed for covert operations and is often deployed in internal honeypot environments to study attack TTPs (Tactics, Techniques, and Procedures).

## 1. Evasion and Persistence Architecture

### AST-Driven Polymorphism Protocol
To defeat static analysis, the builder utilizes a custom source-code mutation protocol:

* **AST-Driven Mutation:** The agent is processed by a custom pre-compiler that mutates the Abstract Syntax Tree (AST) to generate **structurally unique binaries** for every deployment. This defeats Control Flow Graph (CFG) analysis and static signatures, a key step beyond simple file hash mutation.
* **String Context Encryption:** All sensitive strings (e.g., C2 domains, commands) are automatically detected and encrypted at the source-code level and decrypted at runtime using a file-local key.
* **Debug Stripping:** All DWARF tables and symbol information are automatically removed during cross-compilation to severely complicate reverse engineering efforts.

### Operational Stealth
* The agent runs silently as a background process (`-H=windowsgui` flag).
* **Jitter Beacon:** Network heartbeat uses randomized intervals (Base ± Jitter%) to defeat detection rules based on static frequency analysis.

### Self-Contained OPSEC
The **`quit`** command initiates a self-destruct routine, which is critical for operational cleanup:
1.  Removes Windows Registry Run keys used for persistence.
2.  Securely deletes the executable binary from the disk.
3.  Terminates the session and signals the C2 server.

## 2. Streaming Data Telemetry

The core innovation is the streaming telemetry model, designed for high-volume, low-latency exfiltration:
* **Streaming C2:** Data is sent immediately in small, encrypted batches (ZIPs) over a covert TCP channel. This ensures data capture is continuous, even if the session is unexpectedly terminated.
* **Auto-Pillage:** An immediate **Stage 1 Recon Report** is generated upon connection, providing the operator with critical system and file metadata before any manual command is issued.
* **Large File Handling:** Files exceeding a defined threshold (**50MB**) are automatically streamed raw in chunks, bypassing the compression engine to prevent memory spikes and keep CPU load minimal.

## 3. Tooling and Sanitization

The framework includes a custom Builder tool to generate payloads. The code is structured to enforce responsible R&D principles.

**Code Snippet: Stealth Compilation**
*The following sanitized Makefile snippet shows the cross-compilation flags used to ensure a minimal, quiet Windows binary.*
```makefile
# Example of build automation for stealth
build_stealer_windows:
    GOOS=windows GOARCH=amd64 go build -ldflags="-s -w -H=windowsgui" -o dist/agent.exe
````

> **Disclaimer:** *Core OPSEC features, including the encryption/decryption keys, C2 IP/Port configuration, and the full multi-threaded session management logic, are omitted from this public documentation to prevent weaponization and maintain responsible disclosure standards. For the technical mechanism behind the polymorphic build, please refer to the dedicated research: [Go AST Polymorphic Engine](https://ieri.sh/research/building-a-polymorphic-engine/).*

-----

*This tool is for educational research and authorized internal threat simulation only.*
