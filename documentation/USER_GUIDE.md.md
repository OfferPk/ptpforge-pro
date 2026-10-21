\# PTPForge Pro \- User Guide & CI/CD Telemetry Pipeline Manual

\#\# 1\. Web Studio Quickstart

1\. Open \`source-code/index.html\` in any modern web browser.  
2\. Select an industry preset from the top dropdown:  
   \- \*\*5G Open RAN (ITU-T G.8275.1) \[Grade A+\]\*\*: Telecom profile with Boundary Clock, domain 24, Two-Step, hardware timestamping, 12.5ns fiber asymmetry, MaxTE 1500ns.  
   \- \*\*Financial HFT (MiFID II / Arista) \[Grade A+\]\*\*: Arista EOS timing fabric for algorithmic trading, domain 0, 100ns precision, MiFID II compliance.  
   \- \*\*Vulnerable Software Timing \[Grade F\]\*\*: Uncalibrated software timestamping, domain 0 mismatch, missing asymmetry, unconstrained BMCA.  
3\. Review the \*\*10-Point Zero-Trust Scorecard\*\* to inspect profile/domain alignment, asymmetry calibration, and hardware timestamping.  
4\. Modify PTP parameters (Profile, Domain, Step Mode, Timestamps \$t\_1..t\_4\$, Asymmetry Delta, Role Locking).  
5\. Inspect the generated \*\*IEEE 1588-2019 Binary Message Frame & Hex Dump\*\* with byte offsets and decoded field markers.  
6\. Switch synthesizer tabs to export production configurations:  
   \- \*\*Arista EOS\*\*: \`ptp mode boundary\`, \`ptp profile g8275.1\`, \`ptp domain 24\`, \`ptp priority1/2\`, \`ptp asymmetry correction\`.  
   \- \*\*Cisco Nexus NX-OS\*\*: \`feature ptp\`, \`ptp domain 24\`, \`ptp transport-layer ethernet\`.  
   \- \*\*Juniper Junos\*\*: \`set protocols ptp boundary-clock\`, \`set protocols ptp profile g8275-1\`.  
   \- \*\*Linux ptp4l / phc2sys\*\*: \`/etc/linuxptp/ptp4l.conf\` configuration file.  
   \- \*\*Prometheus PTP Exporter\*\*: Standalone Python exporter script for port \`:9128\`.

\---

\#\# 2\. DevSecOps CLI Tool (\`ptpforge-cli.py\`)

The standalone Python CLI tool requires zero external dependencies and runs on Python 3.8+.

\#\#\# 2.1 Encoding IEEE 1588 PTP Binary Hex Dumps  
\`\`\`bash  
python3 build/ptpforge-cli.py encode-ptp \-p presets/telecom-5g-oran-g8275-1.json \-t SYNC  
\`\`\`

\#\#\# 2.2 Calculating Two-Way Delay & Asymmetry Offset  
\`\`\`bash  
python3 build/ptpforge-cli.py calc-offset \--t1 1000000.0 \--t2 1001250.0 \--t3 2000000.0 \--t4 2001237.5 \--asym 12.5 \--max-te 1500.0  
\`\`\`  
Output:  
\`\`\`  
\=== IEEE 1588 Two-Way Path Delay & Time Error Analysis \===  
Forward Delta (t2-t1):   1250.0 ns  
Reverse Delta (t4-t3):   1237.5 ns  
Round-Trip Delay:        2487.5 ns  
Mean Path Delay:         1243.75 ns  
Raw Master Offset:       6.25 ns  
Fiber Asymmetry (Delta): 12.5 ns  
Corrected Master Offset: 0.0 ns  
Time Error (|Offset|):   0.0 ns  
MaxTE Budget:            1500.0 ns  
Compliance Status:       PASSED (Phase Locked)  
\`\`\`

\#\#\# 2.3 Running CI/CD Pipeline Telemetry Audits  
\`\`\`bash  
\# Clean production configuration \-\> Exits code 0  
python3 build/ptpforge-cli.py audit \-p presets/telecom-5g-oran-g8275-1.json \--strict

\# Insecure/fragile configuration \-\> Exits code 1 (Pipeline Fails)  
python3 build/ptpforge-cli.py audit \-p presets/vulnerable-drift-ntp-fallback.json \--strict  
\`\`\`

\#\#\# 2.4 Synthesizing Switch Configurations  
\`\`\`bash  
python3 build/ptpforge-cli.py synthesize \-p presets/telecom-5g-oran-g8275-1.json \-f arista  
python3 build/ptpforge-cli.py synthesize \-p presets/telecom-5g-oran-g8275-1.json \-f linux  
\`\`\`

\---

\#\# 3\. GitHub Actions CI/CD Pipeline Gate

Add this workflow to your NetDevOps repository (\`.github/workflows/ptp-audit-gate.yml\`):

\`\`\`yaml  
name: Zero-Trust PTP Timing CI/CD Gate

on:  
  pull\_request:  
    paths:  
      \- 'ptp/\*\*'  
      \- 'timing/\*\*'  
      \- 'synchronization/\*\*'

jobs:  
  ptp-audit:  
    runs-on: ubuntu-latest  
    steps:  
      \- name: Checkout Code  
        uses: actions/checkout@v4

      \- name: Set up Python  
        uses: actions/setup-python@v5  
        with:  
          python-version: '3.11'

      \- name: Audit PTP Clock Configuration (Strict Gate)  
        run: |  
          python3 build/ptpforge-cli.py audit \-p presets/telecom-5g-oran-g8275-1.json \--strict  
\`\`\`  
