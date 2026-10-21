/\*\*  
 \* PTPForge Pro \- 10-Point Zero-Trust PTP Clock & Fabric Synchronization Auditor  
 \* Evaluates IEEE 1588-2019 & ITU-T G.8275.1 compliance, fiber asymmetry calibration,  
 \* BMCA rogue master defenses, hardware timestamping, and Time Error budgets.  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.PTPSecurityAuditor \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  function auditPTPConfiguration(config) {  
    config \= config || {};  
    var findings \= \[\];  
    var score \= 0;

    var profile \= (config.ptpProfile || 'g8275.1').toLowerCase();  
    var domain \= parseInt(config.domainNumber, 10);  
    if (isNaN(domain)) domain \= 24;

    // Check 1: PTP Profile & Domain Alignment  
    var check1 \= {  
      id: 'PROFILE\_DOMAIN',  
      name: 'PTP Profile & Domain Number Alignment',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (profile \=== 'g8275.1') {  
      if (domain \>= 24 && domain \<= 43\) {  
        check1.points \= 10;  
        check1.status \= 'PASS';  
        check1.description \= 'Compliant ITU-T G.8275.1 telecom profile with designated domain ' \+ domain \+ ' (24–43 range).';  
      } else {  
        check1.points \= 4;  
        check1.status \= 'FAIL';  
        check1.description \= 'G.8275.1 profile requires domain number in 24–43 range (found ' \+ domain \+ ').';  
      }  
    } else if (profile \=== 'default' || profile \=== 'ieee1588') {  
      check1.points \= 10;  
      check1.status \= 'PASS';  
      check1.description \= 'IEEE 1588 default profile configured with domain ' \+ domain \+ '.';  
    } else if (profile \=== 'smpte') {  
      if (domain \=== 127\) {  
        check1.points \= 10;  
        check1.status \= 'PASS';  
        check1.description \= 'SMPTE ST 2059-2 broadcast profile configured with domain 127.';  
      } else {  
        check1.points \= 6;  
        check1.status \= 'WARN';  
        check1.description \= 'SMPTE profile typically uses standard domain 127 (found ' \+ domain \+ ').';  
      }  
    } else {  
      check1.points \= 5;  
      check1.status \= 'WARN';  
      check1.description \= 'Custom PTP profile specified with domain ' \+ domain \+ '.';  
    }  
    score \+= check1.points;  
    findings.push(check1);

    // Check 2: Clock Step Mode Consistency  
    var twoStep \= \!\!config.twoStep;  
    var check2 \= {  
      id: 'STEP\_MODE',  
      name: 'Clock Mode & Follow\_Up Step Consistency',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (twoStep) {  
      check2.points \= 10;  
      check2.status \= 'PASS';  
      check2.description \= 'Two-Step clock active: Precise egress timestamp t1 conveyed via Follow\_Up message.';  
    } else {  
      check2.points \= 8;  
      check2.status \= 'PASS';  
      check2.description \= 'One-Step clock active: Requires PHY-level on-the-fly hardware timestamp insertion.';  
    }  
    score \+= check2.points;  
    findings.push(check2);

    // Check 3: BMCA Rogue Master Defense  
    var bmcaRoleLock \= \!\!config.enableMasterOnlyLock;  
    var prio1 \= parseInt(config.priority1, 10\) || 128;  
    var check3 \= {  
      id: 'BMCA\_SECURITY',  
      name: 'BMCA Rogue Master Defense & Role Locking',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (bmcaRoleLock) {  
      check3.points \= 10;  
      check3.status \= 'PASS';  
      check3.description \= 'Port role locking active (master-only/slave-only): Blocks rogue grandmaster injection.';  
    } else {  
      check3.points \= 2;  
      check3.status \= 'FAIL';  
      check3.description \= 'CRITICAL DEFECT: Ports allow unconstrained BMCA negotiation. Rogue grandmaster can hijack clock tree.';  
    }  
    score \+= check3.points;  
    findings.push(check3);

    // Check 4: Fiber & Transceiver Asymmetry Compensation  
    var hasAsymmetryConfig \= config.fiberAsymmetryNs \!== undefined && parseFloat(config.fiberAsymmetryNs) \!== 0.0;  
    var check4 \= {  
      id: 'ASYMMETRY\_CALIBRATION',  
      name: 'Fiber & Transceiver Asymmetry Calibration',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (hasAsymmetryConfig) {  
      check4.points \= 10;  
      check4.status \= 'PASS';  
      check4.description \= 'Calibrated physical propagation delay asymmetry (' \+ config.fiberAsymmetryNs \+ 'ns) compensates for fiber path mismatches.';  
    } else {  
      check4.points \= 4;  
      check4.status \= 'WARN';  
      check4.description \= 'Uncalibrated fiber asymmetry (0ns). Physical Tx/Rx fiber length deltas will introduce static phase error.';  
    }  
    score \+= check4.points;  
    findings.push(check4);

    // Check 5: Hardware Timestamping Verification  
    var hwTimestamp \= \!\!config.hardwareTimestamping;  
    var check5 \= {  
      id: 'HARDWARE\_TIMESTAMPING',  
      name: 'PHY/MAC Hardware Timestamping Mode',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (hwTimestamp) {  
      check5.points \= 10;  
      check5.status \= 'PASS';  
      check5.description \= 'Hardware PHY/MAC timestamping active: Eliminates OS kernel interrupt and queuing jitter.';  
    } else {  
      check5.points \= 0;  
      check5.status \= 'FAIL';  
      check5.description \= 'CRITICAL FAILURE: Software timestamping in use. Microsecond OS jitter ruins nanosecond accuracy.';  
    }  
    score \+= check5.points;  
    findings.push(check5);

    // Check 6: Message Rate & Interval Calibration  
    var syncInterval \= parseInt(config.logSyncInterval, 10);  
    if (isNaN(syncInterval)) syncInterval \= \-3; // \-3 \= 8 pkts/sec  
    var check6 \= {  
      id: 'MESSAGE\_RATES',  
      name: 'Sync Message Rate & Interval Calibration',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (syncInterval \<= \-3 && syncInterval \>= \-5) {  
      check6.points \= 10;  
      check6.status \= 'PASS';  
      check6.description \= 'High-frequency Sync rate (log ' \+ syncInterval \+ ' \= ' \+ Math.pow(2, \-syncInterval) \+ ' pkts/s) provides fast servo lock.';  
    } else if (syncInterval \<= 0\) {  
      check6.points \= 6;  
      check6.status \= 'WARN';  
      check6.description \= 'Standard Sync rate (log ' \+ syncInterval \+ ' \= ' \+ Math.pow(2, \-syncInterval) \+ ' pkts/s). Recommend higher rate for 5G/HFT.';  
    } else {  
      check6.points \= 2;  
      check6.status \= 'FAIL';  
      check6.description \= 'Low Sync rate (interval \> 1s). Clock phase tracking will suffer severe wander.';  
    }  
    score \+= check6.points;  
    findings.push(check6);

    // Check 7: Transparent/Boundary Clock Path Continuity  
    var clockMode \= (config.clockMode || 'boundary').toLowerCase();  
    var check7 \= {  
      id: 'CLOCK\_CONTINUITY',  
      name: 'Boundary Clock / Transparent Continuity',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (clockMode \=== 'boundary' || clockMode \=== 'bc') {  
      check7.points \= 10;  
      check7.status \= 'PASS';  
      check7.description \= 'Boundary Clock (BC) mode: Terminates and regenerates PTP timestamps, preventing PDV accumulation.';  
    } else if (clockMode \=== 'transparent' || clockMode \=== 'tc') {  
      check7.points \= 8;  
      check7.status \= 'PASS';  
      check7.description \= 'Transparent Clock (TC) mode: Updates packet correction field for transit residence time.';  
    } else {  
      check7.points \= 3;  
      check7.status \= 'WARN';  
      check7.description \= 'Ordinary clock mode configured on transit switch node.';  
    }  
    score \+= check7.points;  
    findings.push(check7);

    // Check 8: Transport Encapsulation Hygiene  
    var transport \= (config.transportEncapsulation || 'ethernet').toLowerCase();  
    var check8 \= {  
      id: 'TRANSPORT\_ENCAPSULATION',  
      name: 'Transport Layer Encapsulation Hygiene',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (profile \=== 'g8275.1') {  
      if (transport \=== 'ethernet' || transport \=== 'l2') {  
        check8.points \= 10;  
        check8.status \= 'PASS';  
        check8.description \= 'Pure Layer 2 Ethernet multicast (01-1B-19-00-00-00 / 01-80-C2-00-00-0E) per ITU-T G.8275.1 mandate.';  
      } else {  
        check8.points \= 0;  
        check8.status \= 'FAIL';  
        check8.description \= 'G.8275.1 strictly forbids IP/UDP encapsulation; must use native IEEE 802.3 Ethernet transport.';  
      }  
    } else {  
      check8.points \= 10;  
      check8.status \= 'PASS';  
      check8.description \= 'Transport encapsulation (' \+ transport.toUpperCase() \+ ') valid for ' \+ profile.toUpperCase() \+ ' profile.';  
    }  
    score \+= check8.points;  
    findings.push(check8);

    // Check 9: GNSS / Grandmaster Redundancy  
    var gmRedundant \= \!\!config.enableGrandmasterRedundancy;  
    var check9 \= {  
      id: 'GM\_REDUNDANCY',  
      name: 'PRTC / GNSS Grandmaster Redundancy',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (gmRedundant) {  
      check9.points \= 10;  
      check9.status \= 'PASS';  
      check9.description \= 'Dual PRTC GNSS grandmasters with automatic hitless failover tracking configured.';  
    } else {  
      check9.points \= 3;  
      check9.status \= 'WARN';  
      check9.description \= 'Single Grandmaster: Single point of failure if GPS antenna or receiver loses lock.';  
    }  
    score \+= check9.points;  
    findings.push(check9);

    // Check 10: Time Error (MaxTE) Threshold Alarms  
    var teAlarm \= \!\!config.enableTimeErrorAlarm;  
    var check10 \= {  
      id: 'TIME\_ERROR\_ALARMS',  
      name: 'Time Error (MaxTE) & Phase Drift Alarms',  
      weight: 10,  
      points: 0,  
      status: 'FAIL',  
      description: ''  
    };  
    if (teAlarm) {  
      check10.points \= 10;  
      check10.status \= 'PASS';  
      check10.description \= 'Real-time phase deviation alarms configured to trigger before MaxTE budget (\>1.5us / 5G) is breached.';  
    } else {  
      check10.points \= 2;  
      check10.status \= 'FAIL';  
      check10.description \= 'No automated phase drift threshold alarms; risk of silent TDD base station frame collisions.';  
    }  
    score \+= check10.points;  
    findings.push(check10);

    // Calculate Grade  
    var grade \= 'F';  
    if (score \>= 90\) grade \= 'A+';  
    else if (score \>= 80\) grade \= 'A';  
    else if (score \>= 70\) grade \= 'B';  
    else if (score \>= 60\) grade \= 'C';  
    else if (score \>= 50\) grade \= 'D';  
    else grade \= 'F';

    return {  
      score: score,  
      grade: grade,  
      passedChecks: findings.filter(function(f) { return f.status \=== 'PASS'; }).length,  
      warningChecks: findings.filter(function(f) { return f.status \=== 'WARN'; }).length,  
      failedChecks: findings.filter(function(f) { return f.status \=== 'FAIL'; }).length,  
      findings: findings  
    };  
  }

  return {  
    auditPTPConfiguration: auditPTPConfiguration  
  };  
}));  
