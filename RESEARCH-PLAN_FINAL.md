# NutureSense — Research Plan

**Project:** Secure IoT-Based Infant Monitoring and Long-Range Transmission of Sensing Data over LoRa
**Type:** CSE Cybersecurity Capstone / Faculty-Guided Project
**Status:** Draft — pending professor confirmation on items marked ⚠️

---

## 1. Background

NutureSense is a secure IoT-based infant monitoring framework developed in the context of infant safety. It combines heterogeneous sensing data — including Wi-Fi Channel State Information (CSI) observations from **RuView**, temperature measurements, and other confirmed infant-monitoring sensor data — with a cybersecurity layer that protects that data before it is transmitted over a long-range wireless link.

Wi-Fi CSI sensing systems such as RuView exploit the fact that a human body or object moving through an indoor space perturbs the Wi-Fi signal in measurable ways. By analyzing amplitude and phase changes in the channel, these systems can detect presence and motion **without cameras or wearable sensors**. RuView is one of the sensing sources in this project — it is the *data source*, not a security mechanism.

Sensing data (CSI-derived observations, temperature, and any other confirmed sensor readings) needs to travel from the monitoring point to a receiving/monitoring system. **LoRa (Long Range)** is a low-power wide-area radio technology suited for exactly this: long-range, low-bandwidth, low-power links, commonly used in IoT deployments where Wi-Fi or cellular is unavailable or too power-hungry.

The gap this project addresses is what happens **between** sensing and reception: infant-monitoring sensor data is sensitive (it can reveal an infant's presence, movement, and physiological state within a space), and wireless links require appropriate protection against threats such as interception, tampering, and spoofing. The security properties of a LoRa-based system depend on the communication architecture and protocols implemented above the physical radio. While LoRa provides the wireless transport, application-level sensing data may require additional authenticated-encryption and replay-protection mechanisms depending on the system design and threat model.

**NutureSense is not a medical device.** It does not diagnose, predict, or prevent any medical condition, and it does not replace medical professionals or clinical monitoring equipment. It is a research and engineering project focused on securely sensing, transmitting, and delivering infant-environment data.

**Confirmed (from provided materials):**
- RuView = Wi-Fi CSI sensing, one data source among several, not an encryption method.
- Temperature is an explicit, confirmed sensing modality alongside RuView.
- LoRa = the long-range communication channel.
- Hardware for LoRa/RuView already exists and is provided by the professor.
- A hybrid/adaptive scheme switching between AES-GCM and ChaCha20-Poly1305 has been suggested.

**Assumed (not yet confirmed) ⚠️:**
- Exact LoRa hardware/module (e.g., SX1276/SX1262-based board, RFM95, etc.)
- Exact RuView hardware/firmware version and its output data format
- Exact identity and models of any "other sensors" beyond RuView and temperature — these remain generic and unconfirmed
- Whether the sensing node(s) and the LoRa transmitter are the same physical device or separate devices connected together
- Whether a gateway/LoRaWAN network server is involved, or if this is point-to-point LoRa

---

## 2. Problem Statement

Infant-monitoring sensor data — CSI-derived observations, temperature, and other confirmed readings — once handed off to a LoRa radio link for long-range transmission, is exposed to standard wireless threats: eavesdropping, tampering, replay, and forgery. Simply encrypting the payload with a single fixed algorithm does not, by itself, guarantee confidentiality, integrity, authenticity, freshness, or resilience against an adaptive attacker — and naively "switching algorithms" for the sake of switching does not inherently improve security unless the switching mechanism is itself authenticated and synchronized between sender and receiver.

**Core problem:** *How can heterogeneous infant-monitoring sensing data be transmitted over a resource-constrained, long-range LoRa link with confidentiality, authentication, integrity, and replay protection, while evaluating whether an adaptive/hybrid cryptographic scheme offers real security or performance advantages over a single fixed authenticated-encryption scheme?*

---

## 3. Motivation

- Infant-monitoring data can reveal sensitive information about an infant's environment, presence, and activity; protecting it in transit matters given the sensitivity of the context, even though this project makes no medical claims.
- LoRa's low bandwidth and power constraints mean cryptographic choices have real, measurable costs (latency, throughput, energy) — this is a genuine engineering trade-off, not just a checkbox.
- The security properties of IoT and LoRa-based deployments can vary depending on the communication architecture, protocol stack, and application-layer implementation. This project provides a concrete case study in designing and measuring authenticated protection for sensing data under resource constraints, within an infant-safety-motivated use case.
- Investigating an adaptive/hybrid scheme (rather than assuming it's automatically better) contributes an evidence-based answer to a design question that is often asserted without proof in industry.
- Combining multiple heterogeneous sensor sources (CSI, temperature, and other confirmed sensors) into a single secure pipeline raises integration questions that a single-sensor project would not surface.

---

## 4. Research Questions

1. **RQ1 (Confidentiality/Integrity):** Can infant-monitoring sensing data be transmitted over LoRa with authenticated encryption (AES-GCM / ChaCha20-Poly1305) while staying within the payload-size and duty-cycle constraints of the platform?
2. **RQ2 (Replay/Forgery resistance):** What mechanism (e.g., nonce + sequence counter + time window) reliably prevents replayed or forged packets from being accepted by the receiver, given LoRa's unreliable, sometimes out-of-order delivery?
3. **RQ3 (Adaptive scheme security):** Does switching between AES-GCM and ChaCha20-Poly1305 in an authenticated, synchronized way provide a measurable security benefit over a single fixed algorithm, or does it only add complexity and attack surface (e.g., downgrade attacks on the negotiation/state-transition step)?
4. **RQ4 (Performance cost):** What is the measurable overhead (latency, throughput, energy, CPU/memory) of each scheme (plain/unencrypted, AES-GCM, ChaCha20-Poly1305, adaptive) on the given hardware?
5. **RQ5 (Attack resilience):** How does each scheme behave under controlled lab demonstrations of eavesdropping, tampering, replay, and forged-packet injection?
6. **RQ6 (Heterogeneous sensor integration):** How can heterogeneous infant-monitoring sensor data, including Wi-Fi CSI-derived observations and temperature measurements, be securely integrated into a common communication and monitoring pipeline while satisfying the resource constraints of the target platform?

---

## 5. Objectives

1. Integrate and represent data from the confirmed infant-monitoring sensors, including temperature and Wi-Fi CSI-derived observations, within the system architecture. *(This objective describes the target/planned integration for the system design; it does not imply that hardware integration has already been completed.)*
2. Understand and characterize the existing RuView + LoRa hardware/firmware setup before modifying anything.
3. Design a packet structure and security layer that provides confidentiality, authentication, integrity, and replay protection for the sensing data (CSI-derived and temperature, plus any other confirmed sensors) sent over LoRa.
4. Implement a baseline (plain, unencrypted) transmission pipeline as a control condition.
5. Implement AES-GCM and ChaCha20-Poly1305 authenticated-encryption pipelines.
6. Design and implement an authenticated, synchronized adaptive/hybrid scheme that switches between the two algorithms under a defined trigger and policy.
7. Build a small, controlled attack suite (eavesdrop capture, tamper injection, replay injection, forged-packet injection) to test each pipeline.
8. Measure and compare communication performance (range, RSSI, SNR, PDR, latency, throughput) and resource cost (CPU, memory, energy, crypto overhead) across all four conditions.
9. Produce a final report/architecture documenting design decisions, results, and trade-offs.

---

## 6. System Architecture

```
                         INFANT / ENVIRONMENT
                                   │
                                   ▼
                     ┌─────────────────────────┐
                     │      SENSING LAYER        │
                     │  RuView / Wi-Fi CSI        │
                     │  Temperature                  │
                     │  Other sensors*                  │
                     └────────────┬─────────────┘
                                  │  Sensor Data
                                  ▼
                     ┌─────────────────────────┐
                     │      SECURITY LAYER      │
                     │  Data Formatting          │
                     │  Key Management            │
                     │  Nonce Management           │
                     │  Replay Protection            │
                     │  Authentication                 │
                     └────────────┬─────────────┘
                                  ▼
                ┌───────────────────────────────┐
                │   AUTHENTICATED ENCRYPTION     │
                │   AES-GCM   OR   ChaCha20-      │
                │            Poly1305             │
                │   (selection governed by the    │
                │    Security Controller, below)  │
                └────────────────┬────────────────┘
                                 │ Secure Data
                                 ▼
                     ┌─────────────────────────┐
                     │   COMMUNICATION LAYER    │
                     │   LoRa / Simulated LoRa   │
                     └────────────┬─────────────┘
                                  │
                             RF CHANNEL
                                  │
                                  ▼
                     ┌─────────────────────────┐
                     │     RECEIVER LAYER        │
                     │  Authentication / Validation │
                     │  Replay Detection                │
                     │  Decryption                          │
                     └────────────┬─────────────┘
                                  ▼
                          Monitoring System
```

*"Other sensors" is intentionally generic. No specific additional sensor models, sampling rates, or interfaces are assumed; the exact sensor set beyond RuView (CSI) and temperature is subject to hardware/professor confirmation ⚠️.

**Adaptive mechanism (sub-architecture):**

```
                    SECURITY CONTROLLER
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
         AES-GCM                 ChaCha20-Poly1305
              │                           │
              └─────────────┬─────────────┘
                            │
                     Secure Protocol
                            │
                            ▼
                           LoRa
```

The Security Controller is responsible for: deciding which algorithm is active, ensuring both sender and receiver agree on the current state (synchronization), authenticating any state-transition message so an attacker cannot force a downgrade, and logging/exposing the decision for evaluation purposes.

**Complete conceptual data flow (multi-sensor):**

```
   Temperature / other sensors*
                \
                 \
            RuView / Wi-Fi CSI
                    ↓
              Data Processing
                    ↓
              Packet Formation
                    ↓
               Authentication
                    ↓
                Encryption
                    ↓
                   LoRa
                    ↓
                 Receiver
                    ↓
          Authentication Check
                    ↓
             Replay Detection
                    ↓
                Decryption
                    ↓
            Monitoring System
```

This is a conceptual architecture only. It does not imply that every sensor or hardware component listed has already been sourced, selected, or integrated; it represents the target data flow the project is designing toward.

### Hardware and Software Components

The project spans both hardware-integrated IoT work and software/cybersecurity research; it is neither a purely software cryptography project nor a purely hardware IoT project.

**Hardware (existing/provided, exact models pending confirmation ⚠️):**
- RuView / Wi-Fi CSI sensing hardware
- Temperature sensing hardware
- LoRa communication hardware (sender side)
- Receiver/gateway hardware, exact architecture depending on the final design
- Additional sensors, subject to confirmation — no specific models assumed

**Software (to be designed/implemented by the project):**
- Sensor-data acquisition
- Data processing/formatting
- Packet construction
- Authentication
- AES-GCM
- ChaCha20-Poly1305
- Key management
- Nonce management
- Replay protection
- Adaptive cryptographic state management
- Receiver-side verification/decryption
- Attack simulations
- Performance/security evaluation

No exact hardware models, firmware versions, or communication interfaces are assumed beyond what is stated above; these remain pending confirmation per Section 9 and Appendix A.

---

## 7. Methodology

**Phase-based, hardware-first approach** (do not design cryptography before understanding the hardware constraints):

1. **Reconnaissance phase** — Inventory the existing RuView, temperature, and LoRa hardware, firmware, and current data path with no modification. Confirm ⚠️-marked assumptions with the professor, including the exact "other sensors" set.
2. **Baseline phase** — Get an unencrypted sensing → LoRa → receiver pipeline working end-to-end for the confirmed sensors. This becomes the performance/security control condition.
3. **Security layer design phase** — Define packet structure (header, sequence number, nonce, ciphertext, authentication tag), key management approach, and replay-window policy at a conceptual level before coding.
4. **Fixed-cipher implementation phase** — Implement AES-GCM, then ChaCha20-Poly1305, each as a drop-in authenticated-encryption layer over the baseline pipeline.
5. **Adaptive scheme phase** — Implement the Security Controller and authenticated state-transition mechanism; define and justify the trigger for switching (e.g., periodic, RSSI/SNR-based, or explicit command), and defend against downgrade/spoofed-switch attacks.
6. **Attack-demonstration phase** — Build controlled, lab-only scripts/tools to eavesdrop, tamper, replay, and forge packets against each pipeline, and record pass/fail behavior.
7. **Measurement phase** — Run each of the four conditions (plain, AES-GCM, ChaCha20-Poly1305, adaptive) through the same test matrix and record all metrics (Section 9).
8. **Analysis and write-up phase** — Compare results against the research questions, and produce the final report and architecture diagrams.

**Guardrail:** No custom/home-grown cryptographic primitives will be designed. Only well-established, peer-reviewed algorithms and libraries are used; the project's contribution is in the *protocol design, adaptive-state security, sensor integration, and evaluation*, not in inventing new cryptography.

---

## 8. Threat Model

**In scope (attacker capabilities assumed):**
- Passive eavesdropping of RF packets within range.
- Active injection of tampered (bit-flipped) packets.
- Replay of previously captured valid packets.
- Injection of entirely forged/fabricated packets.
- Attempts to force or spoof a cryptographic state transition (downgrade attack) in the adaptive scheme.

**Out of scope (assumed, pending professor confirmation ⚠️):**
- Physical compromise of the sensing or receiving hardware (device tampering, key extraction from silicon).
- Side-channel attacks (timing/power analysis) on the cryptographic implementation.
- Jamming/denial-of-service at the RF layer (may be discussed qualitatively but likely not implemented as an experiment).
- Compromise of the professor-provided hardware/firmware supply chain.

**Assets to protect:** confidentiality of infant-monitoring sensing data (CSI-derived, temperature, and any other confirmed sensor readings); integrity and authenticity of each packet; freshness (no accepted replays); integrity of the adaptive scheme's internal state.

**Trust assumptions ⚠️ (to confirm):** how initial key material is provisioned to sender and receiver (pre-shared key vs. some key-exchange step), and whether key rotation is in scope for this project phase.

---

## 9. Experiments and Evaluation

| # | Experiment | Conditions Tested | Metrics |
|---|------------|--------------------|---------|
| 1 | Baseline range/link test | Plain LoRa | Range, RSSI, SNR, PDR |
| 2 | Cryptographic overhead | Plain vs AES-GCM vs ChaCha20-Poly1305 vs Adaptive | Encryption/decryption time, packet size overhead |
| 3 | End-to-end performance | All four conditions | Latency, throughput, PDR, packet loss |
| 4 | Resource cost | All four conditions | CPU usage, memory usage, energy consumption |
| 5 | Eavesdropping demo | Plain vs each cipher | Whether captured payload is human-readable |
| 6 | Tampering demo | Each cipher | Whether tampered packets are correctly rejected |
| 7 | Replay demo | Each cipher | Whether replayed packets are correctly rejected |
| 8 | Forged-packet demo | Each cipher | Whether forged packets are correctly rejected |
| 9 | Adaptive state-transition security | Adaptive scheme only | Whether forced/spoofed transitions are rejected |
| 10 | Adaptive vs fixed comparison | AES-GCM vs ChaCha20-Poly1305 vs Adaptive | Net security benefit vs added latency/complexity |

**Expected results to collect:** tables of raw metric values per condition; success/failure logs for each attack demonstration; overhead percentage of each cipher relative to baseline; a qualitative and quantitative verdict on whether the adaptive scheme is justified.

**Useful graphs/tables for the final report:** PDR vs. distance per scheme; latency/throughput bar chart across the four conditions; energy consumption comparison; a pass/fail matrix of attack demonstrations × schemes; CPU/memory usage over time during encryption bursts.

---

## 10. Expected Contributions

1. A working, documented pipeline that secures multi-source infant-monitoring sensing data (RuView-derived CSI, temperature, and any other confirmed sensors) over a LoRa link with authenticated encryption and replay protection.
2. An authenticated, synchronized adaptive cryptographic scheme design that avoids naive/insecure algorithm-switching, along with an analysis of whether it is actually worth the added complexity.
3. An empirical performance and security comparison of plain LoRa, AES-GCM, ChaCha20-Poly1305, and the adaptive scheme on real (or simulated, if constrained) hardware.
4. A reusable conceptual packet structure and protocol design for securing constrained long-range IoT links carrying heterogeneous sensor data, applicable beyond this specific infant-monitoring use case.
5. A small controlled attack suite (eavesdrop/tamper/replay/forge) usable for future coursework or follow-on projects.

---

## 11. Future Work

- Extend to a full LoRaWAN deployment with proper key-exchange/provisioning instead of pre-shared keys, if the current phase uses pre-shared keys.
- Explore lightweight post-quantum or hybrid classical/PQC schemes suited to constrained LoRa hardware.
- Add side-channel resistance analysis of the cryptographic implementation.
- Extend the adaptive controller to make switching decisions based on live link quality (RSSI/SNR) or detected anomalies rather than a fixed policy.
- Explore multi-node scenarios (multiple sensing nodes reporting to one gateway) and the key-management implications of scaling up.
- Investigate additional confirmed infant-monitoring sensor modalities beyond CSI and temperature, once identified.

---

## Appendix A — Questions to Confirm with the Professor Before Implementation

- Exact LoRa hardware model(s) in use, and whether it's point-to-point LoRa or a LoRaWAN gateway setup.
- Exact RuView hardware/firmware version and the format/rate of data it outputs.
- Exact identity of "other sensors" beyond RuView (CSI) and temperature, if any are planned for this phase.
- Whether the sensing device(s) and the LoRa radio are the same physical board or separate boards linked together (and how).
- How initial cryptographic key material should be provisioned (pre-shared key acceptable, or is a key-exchange step expected?).
- Whether physical/side-channel attacks and RF jamming/DoS are in scope for this project or explicitly out of scope.
- What the trigger/policy for the adaptive scheme should be (fixed schedule, link-quality-based, explicit operator command, etc.), or whether that choice is left to the students.
- Expected deliverable format and depth (working prototype vs. simulation vs. primarily a design + evaluation report).
- Any existing baseline code, libraries, or prior student work on this hardware that should be reused rather than rebuilt.
