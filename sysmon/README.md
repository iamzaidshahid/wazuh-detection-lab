# Sysmon configuration additions

Sysmon only logs file creations for extensions listed in its config, and only logs the network connections it is told to. The detection rules in `rules/` depend on the additions below. They are fragments to add to an existing Sysmon configuration, not a complete config.

Tested with Sysmon v15.15 on a Windows endpoint reporting to a Wazuh manager.

## 1. Ransomware extensions and ransom notes
Used by the ransomware rules (100200 and 100204). Add inside the `<FileCreate onmatch="include">` section:

```xml
<TargetFilename condition="end with">.locked</TargetFilename>
<TargetFilename condition="end with">.encrypted</TargetFilename>
<TargetFilename condition="end with">.enc</TargetFilename>
<TargetFilename condition="end with">.wcry</TargetFilename>
<TargetFilename condition="end with">.wncry</TargetFilename>
<TargetFilename condition="end with">.crypto</TargetFilename>
<TargetFilename condition="end with">.cerber</TargetFilename>
<TargetFilename condition="end with">.locky</TargetFilename>
<TargetFilename condition="end with">.txt</TargetFilename>
<TargetFilename condition="end with">.html</TargetFilename>
<TargetFilename condition="end with">.hta</TargetFilename>
```

## 2. Archive files
Used by the data exfiltration rules (100402 and 100411). Add inside the same `<FileCreate onmatch="include">` section:

```xml
<TargetFilename condition="end with">.zip</TargetFilename>
<TargetFilename condition="end with">.rar</TargetFilename>
<TargetFilename condition="end with">.7z</TargetFilename>
<TargetFilename condition="end with">.tar</TargetFilename>
<TargetFilename condition="end with">.gz</TargetFilename>
```

## 3. Network connections from transfer tools
Used by the data exfiltration rules (100400, 100410 and 100420). Add before the closing `</EventFiltering>` tag:

```xml
<RuleGroup name="" groupRelation="or">
  <NetworkConnect onmatch="include">
    <Image condition="end with">curl.exe</Image>
    <Image condition="end with">wget.exe</Image>
    <Image condition="end with">powershell.exe</Image>
    <Image condition="end with">cmd.exe</Image>
    <Image condition="end with">ftp.exe</Image>
    <Image condition="end with">nc.exe</Image>
    <DestinationPort condition="is">4444</DestinationPort>
    <DestinationPort condition="is">6666</DestinationPort>
    <DestinationPort condition="is">1337</DestinationPort>
    <DestinationPort condition="is">31337</DestinationPort>
    <DestinationPort condition="is">9999</DestinationPort>
    <DestinationPort condition="is">8888</DestinationPort>
  </NetworkConnect>
</RuleGroup>
```

## Reload the configuration
After saving the config file, apply it from an elevated prompt:

```
C:\Windows\Sysmon64.exe -c <path-to-sysmonconfig.xml>
```

## Notes
- Without section 3, connections made by `curl.exe` were not captured by Sysmon in testing.
- Sysmon deduplicates rapid repeated connections to the same destination, so tests of the frequency rules need a short pause between connections.
