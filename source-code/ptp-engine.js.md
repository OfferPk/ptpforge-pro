/\*\*  
 \* PTPForge Pro \- IEEE 1588-2019 (PTPv2.1) & ITU-T G.8275.1 Precision Time Engine  
 \* Models Common PTP Headers, Announce, Sync, Follow\_Up, Delay\_Req/Delay\_Resp bodies,  
 \* Two-Way Delay / Offset Math, and Fiber Asymmetry Calibration.  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.PTPEngine \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  // Helper: Parse Clock Identity (e.g. 00:11:22:ff:fe:33:44:55 or 001122fffe334455) to 8 bytes  
  function parseClockIdentity(idStr) {  
    if (\!idStr || typeof idStr \!== 'string') return \[0, 0, 0, 0, 0, 0, 0, 0\];  
    var clean \= idStr.replace(/\[:-\]/g, '').toLowerCase();  
    var bytes \= \[\];  
    for (var i \= 0; i \< 16; i \+= 2\) {  
      bytes.push(parseInt(clean.substr(i, 2), 16\) || 0);  
    }  
    while (bytes.length \< 8\) bytes.push(0);  
    return bytes.slice(0, 8);  
  }

  // Format 8-byte array as EUI-64 Clock Identity  
  function formatClockIdentity(bytes) {  
    var hex \= \[\];  
    for (var i \= 0; i \< 8; i++) {  
      hex.push(('0' \+ bytes\[i\].toString(16)).slice(-2));  
    }  
    return hex.join(':');  
  }

  // Message Types (IEEE 1588-2019 Table 18\)  
  var MESSAGE\_TYPES \= {  
    SYNC: 0x0,  
    DELAY\_REQ: 0x1,  
    PDELAY\_REQ: 0x2,  
    PDELAY\_RESP: 0x3,  
    FOLLOW\_UP: 0x8,  
    DELAY\_RESP: 0x9,  
    PDELAY\_RESP\_FOLLOW\_UP: 0xa,  
    ANNOUNCE: 0xb,  
    SIGNALING: 0xc,  
    MANAGEMENT: 0xd  
  };

  // Build Common PTP Header (34 Bytes)  
  function buildCommonHeader(params, messageType, bodyLength) {  
    var totalLen \= 34 \+ bodyLength;  
    var domain \= params.domainNumber \!== undefined ? params.domainNumber : 24; // 24 for G.8275.1  
    var twoStep \= \!\!params.twoStep;  
    var seqId \= params.sequenceId \!== undefined ? params.sequenceId : 42;  
    var logInterval \= params.logMessageInterval \!== undefined ? params.logMessageInterval : \-3; // \-3 \= 8 pkts/sec  
    var clockId \= parseClockIdentity(params.clockIdentity || '00:11:22:ff:fe:33:44:55');  
    var portNum \= params.portNumber || 1;

    // Control field legacy mapping  
    var control \= 0x05;  
    if (messageType \=== MESSAGE\_TYPES.SYNC) control \= 0x00;  
    else if (messageType \=== MESSAGE\_TYPES.DELAY\_REQ) control \= 0x01;  
    else if (messageType \=== MESSAGE\_TYPES.FOLLOW\_UP) control \= 0x02;  
    else if (messageType \=== MESSAGE\_TYPES.DELAY\_RESP) control \= 0x03;

    // Flags: Bit 1 (TwoStep: 0x0200), Bit 13 (PTP timescale: 0x0008), Bit 14 (Time Traceable: 0x0004), Bit 15 (Freq: 0x0002)  
    var flagVal \= 0x000e; // PTP timescale \+ Time traceable \+ Freq traceable  
    if (twoStep && (messageType \=== MESSAGE\_TYPES.SYNC)) {  
      flagVal |= 0x0200; // twoStepFlag  
    }

    var hdr \= \[\];  
    // Byte 0: transportSpecific (0) | messageType  
    hdr.push((0x0 \<\< 4\) | (messageType & 0x0f));  
    // Byte 1: versionPTP (2) | minorVersion (1 for IEEE 1588-2019)  
    hdr.push(0x21);  
    // Bytes 2..3: messageLength  
    hdr.push((totalLen \>\> 8\) & 0xff);  
    hdr.push(totalLen & 0xff);  
    // Byte 4: domainNumber  
    hdr.push(domain & 0xff);  
    // Byte 5: reserved  
    hdr.push(0x00);  
    // Bytes 6..7: flags (big-endian)  
    hdr.push((flagVal \>\> 8\) & 0xff);  
    hdr.push(flagVal & 0xff);

    // Bytes 8..15: correctionField (64-bit nanoseconds scaled by 2^16)  
    var corrNs \= params.correctionFieldNs || 0;  
    var scaledCorr \= BigInt(Math.round(corrNs \* 65536));  
    for (var i \= 7; i \>= 0; i--) {  
      hdr.push(Number((scaledCorr \>\> BigInt(i \* 8)) & BigInt(0xff)));  
    }

    // Bytes 16..19: reserved  
    hdr.push(0x00, 0x00, 0x00, 0x00);

    // Bytes 20..27: sourcePortIdentity.clockIdentity (8B)  
    for (var c \= 0; c \< 8; c++) hdr.push(clockId\[c\]);

    // Bytes 28..29: sourcePortIdentity.portNumber (2B)  
    hdr.push((portNum \>\> 8\) & 0xff);  
    hdr.push(portNum & 0xff);

    // Bytes 30..31: sequenceId (2B)  
    hdr.push((seqId \>\> 8\) & 0xff);  
    hdr.push(seqId & 0xff);

    // Byte 32: controlField  
    hdr.push(control & 0xff);

    // Byte 33: logMessageInterval (signed int8)  
    hdr.push((logInterval & 0xff));

    return hdr;  
  }

  // Encode Origin Timestamp (10 Bytes: 6B Seconds \+ 4B Nanoseconds)  
  function encodeTimestamp(seconds, nanoseconds) {  
    var ts \= \[\];  
    var secBig \= BigInt(seconds || Math.floor(Date.now() / 1000));  
    var nsVal \= (nanoseconds || 148201932\) & 0xffffffff;

    for (var i \= 5; i \>= 0; i--) {  
      ts.push(Number((secBig \>\> BigInt(i \* 8)) & BigInt(0xff)));  
    }  
    ts.push((nsVal \>\> 24\) & 0xff);  
    ts.push((nsVal \>\> 16\) & 0xff);  
    ts.push((nsVal \>\> 8\) & 0xff);  
    ts.push(nsVal & 0xff);  
    return ts;  
  }

  // Assemble Complete PTP Message Frame  
  function buildPTPFrame(params) {  
    params \= params || {};  
    var msgTypeStr \= (params.messageType || 'SYNC').toUpperCase();  
    var msgType \= MESSAGE\_TYPES\[msgTypeStr\] \!== undefined ? MESSAGE\_TYPES\[msgTypeStr\] : MESSAGE\_TYPES.SYNC;

    var bodyBytes \= \[\];

    if (msgType \=== MESSAGE\_TYPES.SYNC || msgType \=== MESSAGE\_TYPES.FOLLOW\_UP) {  
      // 10 Bytes originTimestamp  
      bodyBytes \= encodeTimestamp(params.seconds, params.nanoseconds);  
    } else if (msgType \=== MESSAGE\_TYPES.DELAY\_REQ) {  
      // 10 Bytes originTimestamp  
      bodyBytes \= encodeTimestamp(params.seconds, params.nanoseconds);  
    } else if (msgType \=== MESSAGE\_TYPES.DELAY\_RESP) {  
      // 10 Bytes receiveTimestamp \+ 10 Bytes requestingPortIdentity  
      var rxTs \= encodeTimestamp(params.seconds, params.nanoseconds);  
      bodyBytes \= bodyBytes.concat(rxTs);  
      var reqClock \= parseClockIdentity(params.requestingClockIdentity || params.clockIdentity);  
      bodyBytes \= bodyBytes.concat(reqClock);  
      bodyBytes.push(((params.portNumber || 1\) \>\> 8\) & 0xff);  
      bodyBytes.push((params.portNumber || 1\) & 0xff);  
    } else if (msgType \=== MESSAGE\_TYPES.ANNOUNCE) {  
      // 10B originTimestamp  
      bodyBytes \= bodyBytes.concat(encodeTimestamp(params.seconds, params.nanoseconds));  
      // 2B currentUtcOffset  
      var utcOffset \= params.currentUtcOffset || 37;  
      bodyBytes.push((utcOffset \>\> 8\) & 0xff, utcOffset & 0xff);  
      // 1B reserved  
      bodyBytes.push(0x00);  
      // 1B grandmasterPriority1  
      bodyBytes.push((params.priority1 || 128\) & 0xff);  
      // 4B grandmasterClockQuality: Class (1B), Accuracy (1B), Variance (2B)  
      bodyBytes.push((params.clockClass || 6\) & 0xff); // 6 \= PRTC locked to GNSS  
      bodyBytes.push((params.clockAccuracy || 0x20) & 0xff); // 0x20 \= \<25ns  
      bodyBytes.push(0x40, 0x00); // Variance  
      // 1B grandmasterPriority2  
      bodyBytes.push((params.priority2 || 128\) & 0xff);  
      // 8B grandmasterIdentity  
      var gmId \= parseClockIdentity(params.grandmasterIdentity || params.clockIdentity);  
      bodyBytes \= bodyBytes.concat(gmId);  
      // 2B stepsRemoved  
      bodyBytes.push(0x00, 0x00);  
      // 1B timeSource  
      bodyBytes.push(0x20); // 0x20 \= GPS / GNSS  
    }

    var hdrBytes \= buildCommonHeader(params, msgType, bodyBytes.length);  
    var fullFrame \= hdrBytes.concat(bodyBytes);

    return {  
      bytes: fullFrame,  
      messageType: msgTypeStr,  
      totalLength: fullFrame.length,  
      headerLength: hdrBytes.length,  
      bodyLength: bodyBytes.length,  
      domain: params.domainNumber || 24  
    };  
  }

  // Calculate Two-Way Delay, Offset from Master, and Asymmetry Correction  
  function computeTwoWayDelay(params) {  
    params \= params || {};  
    // All timestamps in nanoseconds relative to an epoch second  
    var t1 \= parseFloat(params.t1\_sync\_egress\_ns) || 1000000.0;  
    var t2 \= parseFloat(params.t2\_sync\_ingress\_ns) || 1001250.0;  
    var t3 \= parseFloat(params.t3\_dreq\_egress\_ns) || 2000000.0;  
    var t4 \= parseFloat(params.t4\_dreq\_ingress\_ns) || 2001230.0;  
    var corrNs \= parseFloat(params.correctionFieldNs) || 0.0;  
    var fiberAsymmetryNs \= parseFloat(params.fiberAsymmetryNs) || 0.0; // Positive means forward path \> reverse path

    // IEEE 1588 Two-Way Formulas:  
    // Round-trip delay \= (t2 \- t1) \+ (t4 \- t3)  
    var forwardDelta \= t2 \- t1;  
    var reverseDelta \= t4 \- t3;  
    var roundTripDelay \= forwardDelta \+ reverseDelta \- corrNs;  
    var meanPathDelay \= roundTripDelay / 2.0;

    // Raw Offset without asymmetry:  
    // Offset \= ((t2 \- t1) \- (t4 \- t3) \- corrNs) / 2 \= (t2 \- t1) \- meanPathDelay  
    var rawOffset \= forwardDelta \- meanPathDelay;

    // Calibrated Offset correcting for fiber asymmetry Delta:  
    // Corrected Offset \= rawOffset \- (Delta / 2\)  
    var correctedOffset \= rawOffset \- (fiberAsymmetryNs / 2.0);  
    var timeErrorNs \= Math.abs(correctedOffset);

    // Profile MaxTE Limit:  
    // G.8275.1 Class C/D: 1500 ns  
    // Financial MiFID II: 100,000 ns (100 us) or 1,000 ns (1 us)  
    var maxAllowedTeNs \= parseFloat(params.maxAllowedTeNs) || 1500.0;  
    var isCompliant \= timeErrorNs \<= maxAllowedTeNs;

    var status \= 'SAFE';  
    var statusMessage \= 'PTP clock phase offset is within strict profile limits.';

    if (\!isCompliant) {  
      status \= 'OUT\_OF\_BOUNDS';  
      statusMessage \= 'CRITICAL: Time Error (' \+ timeErrorNs.toFixed(1) \+ 'ns) exceeds profile limit (' \+ maxAllowedTeNs \+ 'ns)\!';  
    } else if (timeErrorNs \> maxAllowedTeNs \* 0.7) {  
      status \= 'DRIFT\_WARNING';  
      statusMessage \= 'WARNING: Phase error approaching threshold (\>70% of budget).';  
    }

    return {  
      t1: t1,  
      t2: t2,  
      t3: t3,  
      t4: t4,  
      forwardDeltaNs: forwardDelta,  
      reverseDeltaNs: reverseDelta,  
      roundTripDelayNs: roundTripDelay,  
      meanPathDelayNs: parseFloat(meanPathDelay.toFixed(2)),  
      rawOffsetNs: parseFloat(rawOffset.toFixed(2)),  
      fiberAsymmetryNs: fiberAsymmetryNs,  
      correctedOffsetNs: parseFloat(correctedOffset.toFixed(2)),  
      timeErrorNs: parseFloat(timeErrorNs.toFixed(2)),  
      maxAllowedTeNs: maxAllowedTeNs,  
      isCompliant: isCompliant,  
      status: status,  
      statusMessage: statusMessage  
    };  
  }

  // Format bytes into Wireshark / hexdump format  
  function formatHexDump(bytes) {  
    var lines \= \[\];  
    for (var i \= 0; i \< bytes.length; i \+= 16\) {  
      var chunk \= bytes.slice(i, i \+ 16);  
      var offsetHex \= ('0000' \+ i.toString(16)).slice(-4);  
      var hexParts \= \[\];  
      var asciiParts \= \[\];

      for (var j \= 0; j \< 16; j++) {  
        if (j \< chunk.length) {  
          var b \= chunk\[j\];  
          hexParts.push(('0' \+ b.toString(16)).slice(-2));  
          asciiParts.push((b \>= 32 && b \<= 126\) ? String.fromCharCode(b) : '.');  
        } else {  
          hexParts.push('  ');  
          asciiParts.push(' ');  
        }  
      }

      var hexFormatted \= hexParts.slice(0, 8).join(' ') \+ '  ' \+ hexParts.slice(8).join(' ');  
      lines.push(offsetHex \+ ':  ' \+ hexFormatted \+ '  |' \+ asciiParts.join('') \+ '|');  
    }  
    return lines.join('\\n');  
  }

  return {  
    MESSAGE\_TYPES: MESSAGE\_TYPES,  
    parseClockIdentity: parseClockIdentity,  
    formatClockIdentity: formatClockIdentity,  
    buildPTPFrame: buildPTPFrame,  
    computeTwoWayDelay: computeTwoWayDelay,  
    formatHexDump: formatHexDump  
  };  
}));  
