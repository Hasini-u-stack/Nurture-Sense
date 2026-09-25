# NutureSense

## A Secure IoT-Based Infant Monitoring and Long-Range Communication Framework

> Cybersecurity / IoT Research Project  
> Domain: Cybersecurity · IoT · Infant Monitoring · Wireless Sensing · Cryptography

---

## Overview

NutureSense is a research-oriented IoT and cybersecurity framework designed
for continuous monitoring of infant-related physiological and environmental
parameters and the secure transmission of the resulting data over a
long-range wireless communication system.

The system is intended to support infant safety monitoring in contexts
where continuous observation of physiological and environmental conditions
may be important, including research related to Sudden Infant Death
Syndrome (SIDS). NutureSense is not intended to diagnose, predict, or
prevent SIDS. Instead, the project focuses on developing a secure
monitoring and communication architecture capable of collecting relevant
data and delivering it reliably to a receiving system.

The sensing layer is designed around RuView and Wi-Fi Channel State
Information (CSI), which can be used to derive information about physical
presence and activity. Additional sensing components, such as temperature
and other infant or environmental parameters, can be incorporated into the
system as the hardware architecture is finalized.

The collected information is passed through a cybersecurity layer before
being transmitted over a long-range LoRa communication link.

The security layer investigates authenticated cryptographic mechanisms,
including AES-GCM and ChaCha20-Poly1305, together with authentication,
integrity protection, replay protection, and secure cryptographic state
management.

The initial development phase is being implemented as a software-only
prototype using simulated sensing data and a simulated LoRa communication
channel. Hardware integration will be performed once the target sensing
and LoRa hardware configuration is confirmed.

---

## Research Motivation

Continuous infant monitoring can involve multiple physiological and
environmental parameters whose changes may be important for caregivers
or monitoring systems.

However, collecting monitoring data is only one part of the problem.

When sensitive sensing information is transmitted through a wireless
communication system, the communication channel introduces cybersecurity
considerations.

An unauthorized party may potentially attempt to:

- Observe transmitted information
- Modify transmitted packets
- Inject forged packets
- Replay previously captured packets
- Manipulate packet metadata
- Interfere with cryptographic state transitions
- Disrupt communication between sensing and receiving systems

This creates a fundamental requirement:

> **Infant monitoring data should not only be collected, but should also
> be transmitted through a communication architecture that provides
> appropriate confidentiality, integrity, authentication, and replay
> protection.**

NutureSense therefore approaches the problem by combining sensing,
secure communication, and long-range wireless transmission into a
single research framework.

---

## Problem Statement

Infant monitoring systems may generate sensitive physiological and
environmental information such as temperature, presence, movement, and
other parameters.

When this information is transmitted wirelessly, protecting the
communication channel becomes important.

A monitoring system that simply transmits raw information may be exposed
to threats such as:

```text
Sensing Data
     |
     v
Wireless Transmission
     |
     v
Potential Interception
     |
     +----> Data Observation
     |
     +----> Packet Tampering
     |
     +----> Packet Forgery
     |
     +----> Replay
     |
     v
Compromised Monitoring Information
