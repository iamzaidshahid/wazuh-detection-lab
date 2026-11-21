# Ransomware detection tests

Commands to trigger each ransomware rule on a Windows test machine with the Wazuh agent and Sysmon installed (see `sysmon/`). After each test, open the Wazuh dashboard and filter by the rule ID shown.

These create empty files and echo commands to simulate the behavior. They do not encrypt anything and do not touch shadow copies. Run them in an elevated PowerShell on a lab machine only.

### Rule 100200: Ransomware Extension File

```powershell
# Run in Windows PowerShell (Admin):
New-Item -Path "$env:TEMP\testfile.locked" -ItemType File -Force
# Check dashboard: rule.id: 100200
# Expected: Alert fires with description 'File created with known ransomware extension'
```

### Rule 100201: Executable Dropped in Suspicious Folder

```powershell
# Run in Windows PowerShell (Admin):
New-Item -Path "$env:TEMP\malware_test.exe" -ItemType File -Force
# Check dashboard: rule.id: 100201
# Expected: Alert fires with description 'Executable dropped in folder commonly used by malware'
```

### Rule 100202: PowerShell File Deletion

```powershell
# Run in Windows PowerShell (Admin):
powershell -Command "Remove-Item '$env:TEMP\testfile.locked' -Force -ErrorAction SilentlyContinue"
# Check dashboard: rule.id: 100202
# Expected: Alert fires with description 'PowerShell deleting files'
```

### Rule 100203: Shadow Copy Deletion

```powershell
# Run in Windows PowerShell (Admin) - SAFE: only echoes the command, doesn't delete:
cmd /c "echo vssadmin delete shadows /all /quiet"
# Check dashboard: rule.id: 100203
# Expected: Alert fires with description 'Shadow copy deletion attempt'
```

### Rule 100204: Ransom Note Dropped

```powershell
# Run in Windows PowerShell (Admin):
New-Item -Path "$env:TEMP\README.txt" -ItemType File -Force
# Check dashboard: rule.id: 100204
# Expected: Alert fires with description 'Possible ransom note file created'
```

### Rule 100210: Mass File Encryption

```powershell
# Run in Windows PowerShell (Admin):
1..15 | ForEach-Object {
New-Item -Path "$env:TEMP\testfile_$_.locked" -ItemType File -Force
Start-Sleep -Milliseconds 200
}
# Check dashboard: rule.id: 100210
# Expected: Alert fires after 10+ files created rapidly by same process
```

### Rule 100211: Mass File Deletion

```powershell
# Run in Windows PowerShell (Admin):
1..8 | ForEach-Object {
powershell -Command "Remove-Item '$env:TEMP\testfile_$_.locked' -Force -ErrorAction SilentlyContinue"
Start-Sleep -Milliseconds 200
}
# Check dashboard: rule.id: 100211
# Expected: Alert fires after 5+ deletions rapidly (also triggers 100220)
```

### Rule 100212: Multiple Ransom Notes

```powershell
# Run in Windows PowerShell (Admin):
$folders = @("$env:TEMP", "$env:USERPROFILE\Desktop",
"$env:USERPROFILE\Documents", "$env:USERPROFILE\Downloads")
$folders | ForEach-Object {
New-Item -Path "$_\README.txt" -ItemType File -Force
Start-Sleep -Milliseconds 200
}
# Check dashboard: rule.id: 100212
# Expected: Alert fires after 4+ ransom notes dropped across folders
```

### Rule 100221: Shadow Deletion + Mass Encryption

```powershell
# Step 1 - Shadow copy deletion (Run first):
cmd /c "echo vssadmin delete shadows /all /quiet"
# Step 2 - Mass encryption immediately after:
1..15 | ForEach-Object {
New-Item -Path "$env:TEMP\testfile3_$_.locked" -ItemType File -Force
Start-Sleep -Milliseconds 200
}
# Check dashboard: rule.id: 100221
# Expected: CRITICAL alert for shadow deletion combined with mass encryption
```

### Rule 100222: Mass Encryption + Ransom Note

```powershell
# Run TWICE in sequence (frequency=2 requires pattern to repeat):
# Step 1 - Mass encryption:
1..15 | ForEach-Object {
New-Item -Path "$env:TEMP\testfile4_$_.locked" -ItemType File -Force
Start-Sleep -Milliseconds 200
}
# Step 2 - Drop ransom note immediately:
New-Item -Path "$env:TEMP\README.txt" -ItemType File -Force
# Repeat both steps once more within 120 seconds
# Check dashboard: rule.id: 100222
# Expected: CRITICAL alert for mass encryption + ransom note combination
```
