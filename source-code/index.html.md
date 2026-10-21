IEEE 1588 • G.8275.1

# 

PTPForge Pro

v1.0.0  
IEEE 1588-2019 (PTPv2.1) & Telecom Timing Profile Studio  
Preset: 5G Open RAN (ITU-T G.8275.1) \[Grade A+\] Preset: Financial HFT (MiFID II / Arista) \[Grade A+\] Preset: Vulnerable Software Timing \[Grade F\] Copy Config Export JSON

## 

1\. PTP Profile & Clock Architecture

PTP Profile ITU-T G.8275.1 (Telecom Full Timing / L2) IEEE 1588 Default (L3 / UDP) SMPTE ST 2059-2 (Broadcast Media)  
Domain Number

Clock Node Mode Boundary Clock (BC) Transparent Clock (TC) Ordinary Clock (OC)  
Transport Layer IEEE 802.3 Ethernet Multicast IPv4 / UDP (Ports 319/320)

Clock Identity (EUI-64)  
Port Number  
Enable Two-Step Clock (Precise t1 via Follow\_Up Message) Hardware PHY / MAC Timestamping Active

## 

2\. BMCA Rogue Master & Asymmetry Defense

Priority 1 (BMCA)  
Priority 2 (BMCA)

Log Sync Interval  
Fiber Asymmetry Delta (ns)  
Port Role Locking Active (Master-Only / Not-Slave) Dual PRTC / GNSS Grandmaster Redundancy Automated Time Error (MaxTE) Phase Drift Alarms

## 

3\. Two-Way Delay Timestamps (Nanoseconds)

t1: Sync Egress (GM)  
t2: Sync Ingress (Slave)

t3: Delay\_Req Egress (Slave)  
t4: Delay\_Req Ingress (GM)  
MaxTE Budget Threshold (ns)

A+

#### 

Zero-Trust PTP Score: 100 / 100

ITU-T G.8275.1 Domain 24 • Boundary Clock • Hardware Timestamped • Asymmetry Calibrated

**Time Error (TE) Phase Offset vs Budget** **Offset: 0.00 ns**  
Mean Delay: **1243.75 ns** Asymmetry Delta: **12.5 ns** MaxTE Budget: **1500 ns**

## 

10-Point Zero-Trust PTP Clock Scorecard

## 

IEEE 1588-2019 Binary Frame & Hex Dump  
Sync Frame | Total: 44B

// Loading hex dump...

Arista EOS Cisco Nexus NX-OS Juniper Junos Linux ptp4l / phc2sys Prometheus Exporter  
\! Generating configuration...