---
title: "Case Study: Engineering Multi-Tenant Security for a CTF Wargames Platform"
date: 2025-11-27
draft: false
description: "A technical review of the Bastioni platform architecture, detailing the use of Split-SSH, user isolation, and intentional flaws for security training."
tags: ["Infrastructure", "Hardening", "Linux", "CTFd", "Engineering"]
---

# Bastioni Platform: Multi-Tenant Hardening Case Study

## Overview
Bastioni is a hybrid wargames platform built for scenario-based cybersecurity training. The primary technical challenge was engineering a secure, multi-tenant Linux environment where players (tenants) could exploit vulnerabilities against adjacent targets without compromising the entire system or affecting other players.

## 1. Isolation Architecture (Defense in Depth)

The platform implements three core layers of isolation to maintain stability and prevent cross-user pivoting:

### A. Split-SSH Access Control
Access is segregated at the network port level:
* **Port 22:** Restricted to administrators only (key-based authentication).
* **Port 2222:** Dedicated to players only (password-based authentication via the `game_users` group).
* **Defense:** This ensures administrative traffic is isolated from potential player brute-force attempts or protocol abuses, and allows separate Fail2Ban rules for each service.

### B. Process and File Hiding
The environment is hardened using system-level controls:
* **`hidepid=2`:** Ensures that players cannot see processes belonging to other users or challenge levels, preventing simple scanning for running exploits or user IDs.
* **`chmod 700`:** Enforced across all user home directories, ensuring strict privacy for player files and challenge assets.

### C. Containerized Boot-to-Root Scenarios
For high-risk challenges (e.g., the **Bravo** category), a Docker-based architecture is used.
* The vulnerable target is run in an ephemeral container, isolated from the host OS.
* **Self-Healing:** A nightly reset script automatically tears down and redeploys the container, guaranteeing a fresh, known-good environment for every session.

## 2. The Progressive Scenario Mechanic

The platform uses a **Flag = Password** progression, forcing users to fully compromise a system before moving on. This mechanic is implemented via intentional flaws:

* **Alpha (Privilege Escalation):** Exploits a single, deliberate flaw (e.g., SUID binaries owned by the next user) to pivot from `user N` to `user N+1`. The flag is the password for `user N+1`.
* **Charlie (Forensics & Reversing):** Users are restricted to their home directory but are granted specific tool access (e.g., GDB/GEF) via group membership (`tools-gef`), isolating the tooling risk.

## 3. Maintenance Philosophy

The design prioritizes stability and clean re-deployment:
* **User Provisioning:** Access to specialized reversing tools is controlled via simple Linux groups (`sudo usermod -aG tools-gef user`).
* **Challenge Isolation:** Creation of "Alpha" challenges involves setting specific group ownership (`sudo chown Alpha1:Alpha0 /home/Alpha1`) to restrict access to the flag file to only the intended user group.

> **Disclaimer:** *Specific server IP addresses and administrative command sequences are omitted from this public review. This document focuses on the defensive architecture and principles used to manage risk in a hostile multi-tenant environment.*