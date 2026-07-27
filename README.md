<div align="center">
  
<img width="2490" height="780" alt="floodgate-banner" src="https://github.com/user-attachments/assets/dbd708ca-88dd-4051-8002-8cb9862c7eaa" />

### A realistic multi-protocol ICS/OT vulnerable lab for offensive and defensive security training.

*Water treatment plant simulation. Six real protocols. Real attacks. Real fixes.*

<br/>

[![Docker](https://img.shields.io/badge/Docker-required-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![License](https://img.shields.io/badge/License-MIT-22c55e?style=flat-square)](LICENSE)
[![Series](https://img.shields.io/badge/InCTRL-Part%202-7c3aed?style=flat-square)](https://rootsec.me)
[![Mode](https://img.shields.io/badge/Mode-VULNERABLE%20%7C%20HARDENED-ef4444?style=flat-square)]()

<br/>

[![Modbus](https://img.shields.io/badge/Modbus%2FTCP-%3A502-f97316?style=flat-square)]()
[![DNP3](https://img.shields.io/badge/DNP3-%3A20000-f97316?style=flat-square)]()
[![MQTT](https://img.shields.io/badge/MQTT-%3A1883-f97316?style=flat-square)]()
[![OPC-UA](https://img.shields.io/badge/OPC--UA-%3A4840-f97316?style=flat-square)]()
[![BACnet](https://img.shields.io/badge/BACnet%2FIP-%3A47808-f97316?style=flat-square)]()
[![S7comm](https://img.shields.io/badge/S7comm-%3A102-f97316?style=flat-square)]()

</div>

---

## What is Floodgate?

Floodgate is a Docker-based ICS/OT security training lab that simulates a realistic municipal water treatment facility. Every component runs a **real protocol stack** — not a mock, not a stub — against a **live process simulation** with actual physics: close the inlet valve and flow drops to zero, over-chlorinate and the safety alarm fires, lose the transfer pump and the tank floods.

It ships in two modes:

| Mode | Purpose |
|------|---------|
| `VULNERABLE` | Everything open. Default credentials. Anonymous access. No segmentation. This is how most real OT environments actually look. Attack everything. |
| `HARDENED` | Protocol authentication enforced. Firewall rules applied. PLC safety interlocks added. Verify your attacks now fail. |

The point is to attack the vulnerable environment, understand exactly why it worked, then harden it yourself and confirm the fix. The `./challenge.sh` tool walks both sides: run the attack, attempt the fix, verify it holds, reveal the solution if you're stuck.

This lab is the practical companion to the **[InCTRL](https://rootsec.me) blog series** — every attack and defensive technique in Parts 3 through 22 runs against Floodgate.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    floodgate-net  172.16.0.0/24                      │
│                                                                       │
│   ┌─────────────┐    ┌──────────────┐    ┌──────────────────────┐   │
│   │  simulation  │    │   openplc    │    │      web-hmi         │   │
│   │ 172.16.0.11  │◄──►│ 172.16.0.10  │◄──►│   172.16.0.12        │   │
│   │ process      │    │ :502 Modbus  │    │ :8080 SCADA UI       │   │
│   │ physics loop │    │ :8080 PLC IDE│    │ :8888 WebSocket      │   │
│   └─────────────┘    └──────────────┘    └──────────────────────┘   │
│                                                                       │
│   ┌─────────────┐    ┌──────────────┐    ┌──────────────────────┐   │
│   │    mqtt      │    │    opcua     │    │       bacnet         │   │
│   │ 172.16.0.15  │    │ 172.16.0.16  │    │   172.16.0.13        │   │
│   │ :1883 broker │    │ :4840 server │    │ :47808/udp           │   │
│   │ :9001 ws     │    │              │    │                      │   │
│   └─────────────┘    └──────────────┘    └──────────────────────┘   │
│                                                                       │
│   ┌─────────────┐    ┌──────────────┐    ┌──────────────────────┐   │
│   │    dnp3      │    │     s7       │    │     scada-web        │   │
│   │ 172.16.0.14  │    │ 172.16.0.17  │    │   172.16.0.18        │   │
│   │ :20000       │    │ :102 S7comm  │    │ :5000 Flask app      │   │
│   │ outstation   │    │ chlor. skid  │    │ ⚠ intentionally      │   │
│   └─────────────┘    └──────────────┘    │   vulnerable         │   │
│                                           └──────────────────────┘   │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │  attacker  ·  172.16.0.99  ·  Kali + all OT tools           │   │
│   └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Protocols

| Protocol | Container | Port | Real Stack | Simulates |
|----------|-----------|------|------------|-----------|
| Modbus/TCP | `openplc` | 502 | pymodbus 3.x / OpenPLC Runtime | Main treatment plant PLC |
| DNP3 | `dnp3` | 20000 | opendnp3 (C++ bindings) | Remote booster pump station RTU |
| MQTT | `mqtt` | 1883 / 9001 ws | Eclipse Mosquitto | IIoT sensor telemetry network |
| OPC-UA | `opcua` | 4840 | asyncua / open62541 | Historian integration layer |
| BACnet/IP | `bacnet` | 47808 udp | bacpypes3 | Facility building automation |
| S7comm | `s7` | 102 | python-snap7 / snap7 server | Chlorination skid (Siemens SIMATIC) |

---

## Quick Start

**Prerequisites:** Docker 24+, Docker Compose v2, 8 GB RAM, 20 GB disk.

```bash
# Clone the repo
git clone https://github.com/rootsec1/floodgate
cd floodgate

# Start in VULNERABLE mode (default — everything open)
docker compose up -d

# Open the SCADA HMI in your browser
open http://localhost:8080

# Get a shell on the attacker container
docker exec -it floodgate-attacker zsh
```

```bash
# Start in HARDENED mode (all fixes applied)
LAB_MODE=hardened docker compose up -d

# Or switch mode on a running lab
./floodgate.sh mode hardened
./floodgate.sh mode vulnerable
```

---

## Lab Modes

Floodgate ships with every vulnerability enabled. The `./challenge.sh` tool manages both sides:

```bash
# List all available challenges
./challenge.sh list

# Run a specific attack (guided walkthrough)
./challenge.sh attack modbus-setpoint

# Attempt the defensive fix yourself, then verify it holds
./challenge.sh harden modbus-setpoint
./challenge.sh verify modbus-setpoint

# Reveal the exact fix if you're stuck
./challenge.sh solution modbus-setpoint

# Apply all fixes at once (full hardened mode)
./challenge.sh harden --all
```

Each challenge has three artifacts: the attack, the detection rule (Suricata/Zeek), and the fix. The fix operates at the correct layer — sometimes network segmentation, sometimes protocol configuration, sometimes a new ladder logic rung in the PLC program.

---

## Container Overview

| Container | Image | Purpose |
|-----------|-------|---------|
| `openplc` | `openplc/openplc_v3` | Real PLC runtime. Runs the water treatment ladder logic. Modbus/TCP slave on :502. Web IDE on :8080. |
| `simulation` | `floodgate/simulation` | Python process physics engine. Reads actuator states from OpenPLC, computes sensor values, writes back. 1 Hz loop. |
| `web-hmi` | `floodgate/web-hmi` | Animated SVG SCADA dashboard. FastAPI backend, WebSocket to browser. Live P&ID with gauges, trend charts, alarm panel. |
| `mqtt` | `eclipse-mosquitto` | MQTT broker. Anonymous auth on in VULNERABLE mode. IIoT telemetry for pumps, sensors, chemical levels. |
| `opcua` | `floodgate/opcua` | OPC-UA server. SecurityMode=None in VULNERABLE mode. Exposes full plant address space. |
| `bacnet` | `floodgate/bacnet` | BACnet/IP device. Simulates facility HVAC, access control, H2S gas detection. |
| `dnp3` | `floodgate/dnp3` | DNP3 outstation. Simulates remote booster pump station RTU. No Secure Authentication in VULNERABLE mode. |
| `s7` | `floodgate/s7` | S7comm server (snap7). Simulates Siemens SIMATIC on the chlorination skid. No access protection in VULNERABLE mode. |
| `scada-web` | `floodgate/scada-web` | Intentionally vulnerable Flask SCADA app. SQLi in tag search, stored XSS in alarm notes, broken authentication, hardcoded credentials. |
| `attacker` | `floodgate/attacker` | Kali Linux with every OT tool pre-installed. Your attack platform. |

---

## Vulnerabilities

Every vulnerability below exists in `VULNERABLE` mode. Every one has a corresponding hardening challenge.

| # | Layer | Vulnerability | Challenge ID |
|---|-------|--------------|--------------|
| 1 | PLC Logic | No chlorine high-high safety interlock in ladder logic | `plc-no-interlock` |
| 2 | PLC Logic | Setpoint register not clamped — accepts any value | `plc-no-clamp` |
| 3 | PLC Logic | Safety coils directly writable over Modbus | `plc-coil-write` |
| 4 | Modbus | Unauthenticated reads and writes on :502 | `modbus-noauth` |
| 5 | Network | Flat /24 — no segmentation between L1, L2, L3 | `net-flat` |
| 6 | MQTT | Anonymous broker — subscribe to `#` from any client | `mqtt-anon` |
| 7 | MQTT | Command topics writable by any connected client | `mqtt-cmd-write` |
| 8 | OPC-UA | SecurityMode = None — no encryption, no auth | `opcua-nosec` |
| 9 | DNP3 | Direct Operate accepted from any source address | `dnp3-noauth` |
| 10 | BACnet | Priority 1 override accepted — bypasses all manual controls | `bacnet-priority` |
| 11 | S7 | No access protection — Stop/Start/program write unrestricted | `s7-noprotect` |
| 12 | SCADA Web | SQL injection in tag search endpoint | `web-sqli` |
| 13 | SCADA Web | Stored XSS in alarm notes field | `web-xss` |
| 14 | SCADA Web | Broken authentication — predictable session tokens | `web-auth` |
| 15 | SCADA Web | Hardcoded credentials in application config | `web-hardcoded` |
| 16 | OpenPLC | Default credentials (admin / admin) on web IDE | `plc-creds` |

---

## Attack Scenarios

Worked attack walkthroughs are published in the **InCTRL** blog series. Each links to the relevant Floodgate challenge.

| Attack | Protocol | Blog Post | Challenge |
|--------|----------|-----------|-----------|
| Passive Modbus reconnaissance | Modbus/TCP | [Part 3](https://rootsec.me) | — |
| Register enumeration and process mapping | Modbus/TCP | [Part 4](https://rootsec.me) | — |
| Chlorine setpoint manipulation (the Oldsmar) | Modbus/TCP | [Part 5](https://rootsec.me) | `modbus-noauth` |
| ARP poisoning on the OT network | ARP | [Part 6](https://rootsec.me) | `net-flat` |
| Modbus MitM — fake sensor readings to HMI | Modbus/TCP | [Part 7](https://rootsec.me) | `net-flat` |
| S7 DB block read and write | S7comm | [Part 8](https://rootsec.me) | `s7-noprotect` |
| EtherNet/IP tag enumeration | EtherNet/IP | [Part 9](https://rootsec.me) | — |
| OPC-UA address space browsing and write | OPC-UA | [Part 10](https://rootsec.me) | `opcua-nosec` |
| DNP3 Direct Operate command injection | DNP3 | [Part 11](https://rootsec.me) | `dnp3-noauth` |
| MQTT broker takeover via command topics | MQTT | [Part 12](https://rootsec.me) | `mqtt-anon` |
| BACnet priority override — HVAC shutdown | BACnet/IP | [Part 13](https://rootsec.me) | `bacnet-priority` |
| Ladder logic analysis and targeted attack | Modbus/TCP | [Part 14](https://rootsec.me) | `plc-no-interlock` |
| SCADA web SQLi — process data exfiltration | HTTP | [Part 17](https://rootsec.me) | `web-sqli` |

---

## Tools Pre-installed on Attacker Container

```
Modbus      pymodbus, mbprobe, mbrecon, modbus-cli
S7          python-snap7, s7scan
EtherNet/IP pycomm3, cpppo
DNP3        opendnp3-python, aegis
BACnet      bacpypes3, BAC0
MQTT        mosquitto-clients, MQTT Explorer, paho-mqtt
OPC-UA      asyncua, opcua-client-gui
Network     Scapy, Wireshark, tshark, nmap, arp-scan, ettercap
Analysis    Grassmarlin, Zeek, Suricata (detection)
General     Python 3.12, Git, curl, netcat, tmux, zsh
```

---

## Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Docker Engine | 24.0 | Latest |
| Docker Compose | v2.20 | Latest |
| RAM | 6 GB | 8 GB |
| Disk | 15 GB | 25 GB |
| OS | Linux / macOS / Windows (WSL2) | Linux |

> **Windows users:** Docker Desktop with WSL2 backend. All commands run in WSL2 terminal.

---

## Part of InCTRL

Floodgate is the lab environment for the **InCTRL — Attacking OT Environments** blog series at [rootsec.me](https://rootsec.me).

| Part | Topic | Uses Floodgate |
|------|-------|---------------|
| Part 2 | Building Your OT Hacking Lab | Setup guide |
| Parts 3–7 | Modbus mastery | Core attack arc |
| Parts 8–9 | S7comm, EtherNet/IP | Vendor protocols |
| Parts 10–12 | OPC-UA, DNP3, MQTT | ICSGoat + Floodgate |
| Part 13 | BACnet and weird protocols | Floodgate BACnet container |
| Parts 14–18 | Advanced techniques | All containers |
| Part 22 | Defending the process | Hardened mode walkthrough |

---

## Legal

Floodgate is built for **authorized security research, education, and training** in isolated lab environments. Do not connect it to production networks. Do not use techniques learned here against systems you do not own or have explicit written permission to test.

Everything in this lab is intentionally broken. That is the point.

---

<div align="center">

Built for the [InCTRL](https://rootsec.me) series &nbsp;·&nbsp; [rootsec.me](https://rootsec.me) &nbsp;·&nbsp; MIT License

</div>
