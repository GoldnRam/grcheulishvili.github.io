---
title: "Lab Architecture"
description: "The internal engineering environment powering IERI.SH R&D."
layout: full
---

<div style="margin-top: 1rem; margin-bottom: 3rem; text-align: center; font-family: 'JetBrains Mono', monospace;">
  <span style="color: #DC143C;">///</span> INTERNAL R&D TOPOLOGY
</div>

## 1. The Validation Range (Bastion)
**Status:** [INTERNAL / PROXMOX]
**Architecture:** Hybrid (CTFd Container + Hardened VMs)
**Role:**
A live-fire assessment environment running on a segmented Proxmox cluster.
* **Engineering Focus:** Designed to simulate "breach scenarios" safely. Uses `Split-SSH` and `ForceCommand` restrictions to isolate attacker sessions from the hypervisor.
* **Utility:** Serves as the primary testing ground for my offensive tooling and persistence scripts.

## 2. Threat Aggregation Node (CTI)
* **Status:** [LOCAL / DOCKER]
* **Architecture:** Python Collectors / PostgreSQL / Grafana
* **Role:** A custom intelligence pipeline that ingests and filters regional threat feeds.
* **Engineering Focus:** Solves the "Alert Fatigue" problem by normalizing disparate feeds (OTX, ThreatFox) into a single high-fidelity PostgreSQL schema.
* **Utility:** Generates the raw blocklists used in my firewall research.

<div style="margin-top: 3rem; margin-bottom: 2rem; border-top: 1px dashed #444;"></div>

## 3. Future Roadmap: Advanced Telemetry
* **Status:** [DESIGN PHASE]
* **Concept:** Replacing the default CTFd scoreboard logs with a custom ELK Stack to capture granular system-level telemetry (Syscalls, Process Creation).
* **Goal:** To move from "Flag-based" validation to "Behavior-based" analysis, allowing for deeper insight into candidate methodology.