/\*\*  
 \* PTPForge Pro \- UI Orchestration & Event Controller  
 \*/

(function () {  
  'use strict';

  var currentTab \= 'arista';  
  var presets \= {  
    'telecom-5g-oran-g8275-1': {  
      ptpProfile: 'g8275.1',  
      domainNumber: 24,  
      clockMode: 'boundary',  
      twoStep: true,  
      hardwareTimestamping: true,  
      clockIdentity: '00:11:22:ff:fe:33:44:55',  
      portNumber: 1,  
      priority1: 128,  
      priority2: 128,  
      logSyncInterval: \-3,  
      enableMasterOnlyLock: true,  
      fiberAsymmetryNs: 12.5,  
      transportEncapsulation: 'ethernet',  
      enableGrandmasterRedundancy: true,  
      enableTimeErrorAlarm: true,  
      maxAllowedTeNs: 1500.0,  
      t1\_sync\_egress\_ns: 1000000.0,  
      t2\_sync\_ingress\_ns: 1001250.0,  
      t3\_dreq\_egress\_ns: 2000000.0,  
      t4\_dreq\_ingress\_ns: 2001237.5  
    },  
    'financial-hft-mifid2-arista': {  
      ptpProfile: 'default',  
      domainNumber: 0,  
      clockMode: 'boundary',  
      twoStep: true,  
      hardwareTimestamping: true,  
      clockIdentity: '00:50:56:ff:fe:a1:b2:c3',  
      portNumber: 1,  
      priority1: 128,  
      priority2: 128,  
      logSyncInterval: \-4,  
      enableMasterOnlyLock: true,  
      fiberAsymmetryNs: 5.0,  
      transportEncapsulation: 'udp',  
      enableGrandmasterRedundancy: true,  
      enableTimeErrorAlarm: true,  
      maxAllowedTeNs: 1000.0,  
      t1\_sync\_egress\_ns: 5000000.0,  
      t2\_sync\_ingress\_ns: 5000450.0,  
      t3\_dreq\_egress\_ns: 6000000.0,  
      t4\_dreq\_ingress\_ns: 6000445.0  
    },  
    'vulnerable-drift-ntp-fallback': {  
      ptpProfile: 'g8275.1',  
      domainNumber: 0,  
      clockMode: 'ordinary',  
      twoStep: false,  
      hardwareTimestamping: false,  
      clockIdentity: '00:00:00:ff:fe:00:00:00',  
      portNumber: 1,  
      priority1: 255,  
      priority2: 255,  
      logSyncInterval: 1,  
      enableMasterOnlyLock: false,  
      fiberAsymmetryNs: 0.0,  
      transportEncapsulation: 'udp',  
      enableGrandmasterRedundancy: false,  
      enableTimeErrorAlarm: false,  
      maxAllowedTeNs: 1500.0,  
      t1\_sync\_egress\_ns: 1000000.0,  
      t2\_sync\_ingress\_ns: 1008500.0,  
      t3\_dreq\_egress\_ns: 2000000.0,  
      t4\_dreq\_ingress\_ns: 2001200.0  
    }  
  };

  function getFormConfig() {  
    return {  
      ptpProfile: document.getElementById('ptpProfile').value,  
      domainNumber: parseInt(document.getElementById('domainNumber').value, 10\) || 0,  
      clockMode: document.getElementById('clockMode').value,  
      transportEncapsulation: document.getElementById('transportEncapsulation').value,  
      clockIdentity: document.getElementById('clockIdentity').value.trim(),  
      portNumber: parseInt(document.getElementById('portNumber').value, 10\) || 1,  
      twoStep: document.getElementById('twoStep').checked,  
      hardwareTimestamping: document.getElementById('hardwareTimestamping').checked,  
      priority1: parseInt(document.getElementById('priority1').value, 10\) || 128,  
      priority2: parseInt(document.getElementById('priority2').value, 10\) || 128,  
      logSyncInterval: parseInt(document.getElementById('logSyncInterval').value, 10\) || \-3,  
      fiberAsymmetryNs: parseFloat(document.getElementById('fiberAsymmetryNs').value) || 0.0,  
      enableMasterOnlyLock: document.getElementById('enableMasterOnlyLock').checked,  
      enableGrandmasterRedundancy: document.getElementById('enableGrandmasterRedundancy').checked,  
      enableTimeErrorAlarm: document.getElementById('enableTimeErrorAlarm').checked,  
      maxAllowedTeNs: parseFloat(document.getElementById('maxAllowedTeNs').value) || 1500.0,  
      t1\_sync\_egress\_ns: parseFloat(document.getElementById('t1SyncEgress').value) || 1000000.0,  
      t2\_sync\_ingress\_ns: parseFloat(document.getElementById('t2SyncIngress').value) || 1001250.0,  
      t3\_dreq\_egress\_ns: parseFloat(document.getElementById('t3DreqEgress').value) || 2000000.0,  
      t4\_dreq\_ingress\_ns: parseFloat(document.getElementById('t4DreqIngress').value) || 2001237.5  
    };  
  }

  function applyPreset(presetKey) {  
    var p \= presets\[presetKey\];  
    if (\!p) return;

    document.getElementById('ptpProfile').value \= p.ptpProfile;  
    document.getElementById('domainNumber').value \= p.domainNumber;  
    document.getElementById('clockMode').value \= p.clockMode;  
    document.getElementById('transportEncapsulation').value \= p.transportEncapsulation;  
    document.getElementById('clockIdentity').value \= p.clockIdentity;  
    document.getElementById('portNumber').value \= p.portNumber;  
    document.getElementById('twoStep').checked \= p.twoStep;  
    document.getElementById('hardwareTimestamping').checked \= p.hardwareTimestamping;  
    document.getElementById('priority1').value \= p.priority1;  
    document.getElementById('priority2').value \= p.priority2;  
    document.getElementById('logSyncInterval').value \= p.logSyncInterval;  
    document.getElementById('fiberAsymmetryNs').value \= p.fiberAsymmetryNs;  
    document.getElementById('enableMasterOnlyLock').checked \= p.enableMasterOnlyLock;  
    document.getElementById('enableGrandmasterRedundancy').checked \= p.enableGrandmasterRedundancy;  
    document.getElementById('enableTimeErrorAlarm').checked \= p.enableTimeErrorAlarm;  
    document.getElementById('maxAllowedTeNs').value \= p.maxAllowedTeNs;  
    document.getElementById('t1SyncEgress').value \= p.t1\_sync\_egress\_ns;  
    document.getElementById('t2SyncIngress').value \= p.t2\_sync\_ingress\_ns;  
    document.getElementById('t3DreqEgress').value \= p.t3\_dreq\_egress\_ns;  
    document.getElementById('t4DreqIngress').value \= p.t4\_dreq\_ingress\_ns;

    updateStudio();  
  }

  function updateStudio() {  
    var cfg \= getFormConfig();

    // 1\. Run Auditor  
    var audit \= window.PTPSecurityAuditor.auditPTPConfiguration(cfg);  
    var badge \= document.getElementById('scoreBadge');  
    badge.textContent \= audit.grade;  
    badge.className \= 'score-badge ' \+ (  
      audit.grade \=== 'A+' || audit.grade \=== 'A' ? 'grade-a' :  
      audit.grade \=== 'B' ? 'grade-b' :  
      audit.grade \=== 'C' ? 'grade-c' : 'grade-f'  
    );

    document.getElementById('scoreTitle').textContent \= 'Zero-Trust PTP Score: ' \+ audit.score \+ ' / 100 (' \+ audit.grade \+ ')';  
    document.getElementById('scoreSummary').textContent \=  
      audit.passedChecks \+ ' Checks Passed \&bull; ' \+ audit.warningChecks \+ ' Warnings \&bull; ' \+ audit.failedChecks \+ ' Critical Risks';

    // Render 10 Rubric Items  
    var container \= document.getElementById('rubricContainer');  
    container.innerHTML \= '';  
    for (var i \= 0; i \< audit.findings.length; i++) {  
      var item \= audit.findings\[i\];  
      var div \= document.createElement('div');  
      var statusClass \= item.status \=== 'PASS' ? 'pass' : (item.status \=== 'WARN' ? 'warn' : 'fail');  
      var badgeClass \= item.status \=== 'PASS' ? 'status-pass-badge' : (item.status \=== 'WARN' ? 'status-warn-badge' : 'status-fail-badge');  
      div.className \= 'rubric-item ' \+ statusClass;  
      div.innerHTML \= '\<div\>\<strong\>' \+ (i \+ 1\) \+ '. ' \+ item.name \+ '\</strong\> (' \+ item.points \+ '/' \+ item.weight \+ ' pts)\<br\>' \+  
        '\<span style="color:var(--text-secondary);"\>' \+ item.description \+ '\</span\>\</div\>' \+  
        '\<span class="rubric-status ' \+ badgeClass \+ '"\>' \+ item.status \+ '\</span\>';  
      container.appendChild(div);  
    }

    // 2\. Two-Way Delay & Time Error Calculations  
    var teMath \= window.PTPEngine.computeTwoWayDelay(cfg);  
    var teStatusSpan \= document.getElementById('teStatusText');  
    if (teMath.isCompliant) {  
      teStatusSpan.textContent \= 'Offset: ' \+ teMath.correctedOffsetNs \+ ' ns (Compliant)';  
      teStatusSpan.style.color \= '\#34d399';  
    } else {  
      teStatusSpan.textContent \= 'Offset: ' \+ teMath.correctedOffsetNs \+ ' ns (OUT OF BOUNDS)';  
      teStatusSpan.style.color \= '\#f87171';  
    }

    var usedPct \= Math.min(100, Math.max(1, Math.round((teMath.timeErrorNs / teMath.maxAllowedTeNs) \* 100)));  
    document.getElementById('barTeUsed').style.width \= usedPct \+ '%';  
    document.getElementById('barTeBudget').style.width \= (100 \- usedPct) \+ '%';  
    if (\!teMath.isCompliant) {  
      document.getElementById('barTeUsed').style.background \= '\#ef4444';  
    } else {  
      document.getElementById('barTeUsed').style.background \= '\#10b981';  
    }

    document.getElementById('valMeanDelay').textContent \= teMath.meanPathDelayNs \+ ' ns';  
    document.getElementById('valAsymDelta').textContent \= teMath.fiberAsymmetryNs \+ ' ns';  
    document.getElementById('valBudget').textContent \= teMath.maxAllowedTeNs \+ ' ns';

    // 3\. Binary Hex Dump Wireframe (Sync Frame)  
    var frame \= window.PTPEngine.buildPTPFrame({  
      messageType: 'SYNC',  
      domainNumber: cfg.domainNumber,  
      twoStep: cfg.twoStep,  
      clockIdentity: cfg.clockIdentity,  
      portNumber: cfg.portNumber,  
      sequenceId: 42,  
      logMessageInterval: cfg.logSyncInterval,  
      correctionFieldNs: 0.0  
    });  
    var hex \= window.PTPEngine.formatHexDump(frame.bytes);  
    document.getElementById('wireframeMeta').textContent \=  
      frame.messageType \+ ' Frame | Total: ' \+ frame.totalLength \+ 'B (Header: ' \+ frame.headerLength \+ 'B | Body: ' \+ frame.bodyLength \+ 'B)';  
    document.getElementById('hexDumpOutput').textContent \= hex;

    // 4\. Multi-Vendor Synthesizer  
    var code \= '';  
    if (currentTab \=== 'arista') code \= window.PTPConfigSynthesizer.synthesizeAristaEOS(cfg);  
    else if (currentTab \=== 'cisco-nxos') code \= window.PTPConfigSynthesizer.synthesizeCiscoNXOS(cfg);  
    else if (currentTab \=== 'juniper') code \= window.PTPConfigSynthesizer.synthesizeJuniperJunos(cfg);  
    else if (currentTab \=== 'linux') code \= window.PTPConfigSynthesizer.synthesizeLinuxPTP(cfg);  
    else if (currentTab \=== 'prometheus') code \= window.PTPConfigSynthesizer.synthesizePrometheusExporter(cfg);

    document.getElementById('configOutput').textContent \= code;  
  }

  // Event Listeners  
  document.addEventListener('DOMContentLoaded', function () {  
    var inputs \= document.querySelectorAll('input, select');  
    for (var i \= 0; i \< inputs.length; i++) {  
      inputs\[i\].addEventListener('input', updateStudio);  
      inputs\[i\].addEventListener('change', updateStudio);  
    }

    document.getElementById('presetSelect').addEventListener('change', function (e) {  
      applyPreset(e.target.value);  
    });

    var tabBtns \= document.querySelectorAll('.tab-btn');  
    for (var j \= 0; j \< tabBtns.length; j++) {  
      tabBtns\[j\].addEventListener('click', function (e) {  
        for (var k \= 0; k \< tabBtns.length; k++) tabBtns\[k\].classList.remove('active');  
        e.target.classList.add('active');  
        currentTab \= e.target.getAttribute('data-tab');  
        updateStudio();  
      });  
    }

    document.getElementById('copyConfigBtn').addEventListener('click', function () {  
      var text \= document.getElementById('configOutput').textContent;  
      navigator.clipboard.writeText(text).then(function () {  
        var btn \= document.getElementById('copyConfigBtn');  
        btn.textContent \= 'Copied\!';  
        setTimeout(function () { btn.textContent \= 'Copy Config'; }, 2000);  
      });  
    });

    document.getElementById('exportJsonBtn').addEventListener('click', function () {  
      var cfg \= getFormConfig();  
      var jsonStr \= JSON.stringify(cfg, null, 2);  
      var blob \= new Blob(\[jsonStr\], { type: 'application/json' });  
      var url \= URL.createObjectURL(blob);  
      var a \= document.createElement('a');  
      a.href \= url;  
      a.download \= 'ptp-config.json';  
      a.click();  
      URL.revokeObjectURL(url);  
    });

    // Initial render  
    updateStudio();  
  });  
})();  
