| Title                    | Dumping Lsass.exe Memory with MiniDumpWriteDump API       |
|:-------------------------|:------------------|
| **Description**          | Detects the use of MiniDumpWriteDump API for dumping lsass.exe memory in a stealth way. Tools like ProcessHacker and some attacker tradecract use this API found in dbghelp.dll or dbgcore.dll. As an example, SilentTrynity C2 Framework has a module that leverages this API to dump the contents of Lsass.exe and transfer it over the network back to the attacker's machine. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0011_7_windows_sysmon_image_loaded](../Data_Needed/DN_0011_7_windows_sysmon_image_loaded.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Penetration tests</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://docs.microsoft.com/en-us/windows/win32/api/minidumpapiset/nf-minidumpapiset-minidumpwritedump](https://docs.microsoft.com/en-us/windows/win32/api/minidumpapiset/nf-minidumpapiset-minidumpwritedump)</li><li>[https://www.pinvoke.net/default.aspx/dbghelp/MiniDumpWriteDump.html](https://www.pinvoke.net/default.aspx/dbghelp/MiniDumpWriteDump.html)</li><li>[https://medium.com/@fsx30/bypass-edrs-memory-protection-introduction-to-hooking-2efb21acffd6](https://medium.com/@fsx30/bypass-edrs-memory-protection-introduction-to-hooking-2efb21acffd6)</li></ul>  |
| **Author**               | Perez Diego (@darkquassar), oscd.community |


## Detection Rules

### Sigma rule

```
title: Dumping Lsass.exe Memory with MiniDumpWriteDump API
id: dd5ab153-beaa-4315-9647-65abc5f71541
status: experimental
description: Detects the use of MiniDumpWriteDump API for dumping lsass.exe memory in a stealth way. Tools like ProcessHacker and some attacker tradecract use this
    API found in dbghelp.dll or dbgcore.dll. As an example, SilentTrynity C2 Framework has a module that leverages this API to dump the contents of Lsass.exe and
    transfer it over the network back to the attacker's machine.
date: 2019/10/27
modified: 2019/11/13
author: Perez Diego (@darkquassar), oscd.community
references:
    - https://docs.microsoft.com/en-us/windows/win32/api/minidumpapiset/nf-minidumpapiset-minidumpwritedump
    - https://www.pinvoke.net/default.aspx/dbghelp/MiniDumpWriteDump.html
    - https://medium.com/@fsx30/bypass-edrs-memory-protection-introduction-to-hooking-2efb21acffd6
tags:
    - attack.credential_access
    - attack.t1003
logsource:
    product: windows
    service: sysmon
detection:
    signedprocess:
        EventID: 7
        ImageLoaded|endswith:
            - '\dbghelp.dll'
            - '\dbgcore.dll'
        Image|endswith: 
            - '\msbuild.exe'
            - '\cmd.exe'
            - '\svchost.exe'
            - '\rundll32.exe'
            - '\powershell.exe'
            - '\word.exe'
            - '\excel.exe'
            - '\powerpnt.exe'
            - '\outlook.exe'
            - '\monitoringhost.exe'
            - '\wmic.exe'
            - '\msiexec.exe'
            - '\bash.exe'
            - '\wscript.exe'
            - '\cscript.exe'
            - '\mshta.exe'
            - '\regsvr32.exe'
            - '\schtasks.exe'
            - '\dnx.exe'
            - '\regsvcs.exe'
            - '\sc.exe'
            - '\scriptrunner.exe'
    unsignedprocess:
        EventID: 7
        ImageLoaded|endswith:
            - '\dbghelp.dll'
            - '\dbgcore.dll'
        Signed: "FALSE"
    filter:
        Image|contains: 'Visual Studio'
    condition: (signedprocess AND NOT filter) OR (unsignedprocess AND NOT filter)
fields:
    - ComputerName
    - User
    - Image
    - ImageLoaded
falsepositives:
    - Penetration tests
level: critical

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {(((($_.ID -eq "7" -and ($_.message -match "ImageLoaded.*.*\\dbghelp.dll" -or $_.message -match "ImageLoaded.*.*\\dbgcore.dll") -and ($_.message -match "Image.*.*\\msbuild.exe" -or $_.message -match "Image.*.*\\cmd.exe" -or $_.message -match "Image.*.*\\svchost.exe" -or $_.message -match "Image.*.*\\rundll32.exe" -or $_.message -match "Image.*.*\\powershell.exe" -or $_.message -match "Image.*.*\\word.exe" -or $_.message -match "Image.*.*\\excel.exe" -or $_.message -match "Image.*.*\\powerpnt.exe" -or $_.message -match "Image.*.*\\outlook.exe" -or $_.message -match "Image.*.*\\monitoringhost.exe" -or $_.message -match "Image.*.*\\wmic.exe" -or $_.message -match "Image.*.*\\msiexec.exe" -or $_.message -match "Image.*.*\\bash.exe" -or $_.message -match "Image.*.*\\wscript.exe" -or $_.message -match "Image.*.*\\cscript.exe" -or $_.message -match "Image.*.*\\mshta.exe" -or $_.message -match "Image.*.*\\regsvr32.exe" -or $_.message -match "Image.*.*\\schtasks.exe" -or $_.message -match "Image.*.*\\dnx.exe" -or $_.message -match "Image.*.*\\regsvcs.exe" -or $_.message -match "Image.*.*\\sc.exe" -or $_.message -match "Image.*.*\\scriptrunner.exe")) -and  -not ($_.message -match "Image.*.*Visual Studio.*")) -or (($_.ID -eq "7" -and ($_.message -match "ImageLoaded.*.*\\dbghelp.dll" -or $_.message -match "ImageLoaded.*.*\\dbgcore.dll") -and $_.message -match "Signed.*FALSE") -and  -not ($_.message -match "Image.*.*Visual Studio.*")))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND (((winlog.event_id:"7" AND winlog.event_data.ImageLoaded.keyword:(*\\dbghelp.dll OR *\\dbgcore.dll) AND winlog.event_data.Image.keyword:(*\\msbuild.exe OR *\\cmd.exe OR *\\svchost.exe OR *\\rundll32.exe OR *\\powershell.exe OR *\\word.exe OR *\\excel.exe OR *\\powerpnt.exe OR *\\outlook.exe OR *\\monitoringhost.exe OR *\\wmic.exe OR *\\msiexec.exe OR *\\bash.exe OR *\\wscript.exe OR *\\cscript.exe OR *\\mshta.exe OR *\\regsvr32.exe OR *\\schtasks.exe OR *\\dnx.exe OR *\\regsvcs.exe OR *\\sc.exe OR *\\scriptrunner.exe)) AND (NOT (winlog.event_data.Image.keyword:*Visual\ Studio*))) OR ((winlog.event_id:"7" AND winlog.event_data.ImageLoaded.keyword:(*\\dbghelp.dll OR *\\dbgcore.dll) AND Signed:"FALSE") AND (NOT (winlog.event_data.Image.keyword:*Visual\ Studio*)))))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/dd5ab153-beaa-4315-9647-65abc5f71541 <<EOF
{
  "metadata": {
    "title": "Dumping Lsass.exe Memory with MiniDumpWriteDump API",
    "description": "Detects the use of MiniDumpWriteDump API for dumping lsass.exe memory in a stealth way. Tools like ProcessHacker and some attacker tradecract use this API found in dbghelp.dll or dbgcore.dll. As an example, SilentTrynity C2 Framework has a module that leverages this API to dump the contents of Lsass.exe and transfer it over the network back to the attacker's machine.",
    "tags": [
      "attack.credential_access",
      "attack.t1003"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (((winlog.event_id:\"7\" AND winlog.event_data.ImageLoaded.keyword:(*\\\\dbghelp.dll OR *\\\\dbgcore.dll) AND winlog.event_data.Image.keyword:(*\\\\msbuild.exe OR *\\\\cmd.exe OR *\\\\svchost.exe OR *\\\\rundll32.exe OR *\\\\powershell.exe OR *\\\\word.exe OR *\\\\excel.exe OR *\\\\powerpnt.exe OR *\\\\outlook.exe OR *\\\\monitoringhost.exe OR *\\\\wmic.exe OR *\\\\msiexec.exe OR *\\\\bash.exe OR *\\\\wscript.exe OR *\\\\cscript.exe OR *\\\\mshta.exe OR *\\\\regsvr32.exe OR *\\\\schtasks.exe OR *\\\\dnx.exe OR *\\\\regsvcs.exe OR *\\\\sc.exe OR *\\\\scriptrunner.exe)) AND (NOT (winlog.event_data.Image.keyword:*Visual\\ Studio*))) OR ((winlog.event_id:\"7\" AND winlog.event_data.ImageLoaded.keyword:(*\\\\dbghelp.dll OR *\\\\dbgcore.dll) AND Signed:\"FALSE\") AND (NOT (winlog.event_data.Image.keyword:*Visual\\ Studio*)))))"
  },
  "trigger": {
    "schedule": {
      "interval": "30m"
    }
  },
  "input": {
    "search": {
      "request": {
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "must": [
                {
                  "query_string": {
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (((winlog.event_id:\"7\" AND winlog.event_data.ImageLoaded.keyword:(*\\\\dbghelp.dll OR *\\\\dbgcore.dll) AND winlog.event_data.Image.keyword:(*\\\\msbuild.exe OR *\\\\cmd.exe OR *\\\\svchost.exe OR *\\\\rundll32.exe OR *\\\\powershell.exe OR *\\\\word.exe OR *\\\\excel.exe OR *\\\\powerpnt.exe OR *\\\\outlook.exe OR *\\\\monitoringhost.exe OR *\\\\wmic.exe OR *\\\\msiexec.exe OR *\\\\bash.exe OR *\\\\wscript.exe OR *\\\\cscript.exe OR *\\\\mshta.exe OR *\\\\regsvr32.exe OR *\\\\schtasks.exe OR *\\\\dnx.exe OR *\\\\regsvcs.exe OR *\\\\sc.exe OR *\\\\scriptrunner.exe)) AND (NOT (winlog.event_data.Image.keyword:*Visual\\ Studio*))) OR ((winlog.event_id:\"7\" AND winlog.event_data.ImageLoaded.keyword:(*\\\\dbghelp.dll OR *\\\\dbgcore.dll) AND Signed:\"FALSE\") AND (NOT (winlog.event_data.Image.keyword:*Visual\\ Studio*)))))",
                    "analyze_wildcard": true
                  }
                }
              ],
              "filter": {
                "range": {
                  "timestamp": {
                    "gte": "now-30m/m"
                  }
                }
              }
            }
          }
        },
        "indices": [
          "winlogbeat-*"
        ]
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "not_eq": 0
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "to": "root@localhost",
        "subject": "Sigma Rule 'Dumping Lsass.exe Memory with MiniDumpWriteDump API'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\nComputerName = {{_source.ComputerName}}\n        User = {{_source.User}}\n       Image = {{_source.Image}}\n ImageLoaded = {{_source.ImageLoaded}}================================================================================\n{{/ctx.payload.hits.hits}}",
        "attachments": {
          "data.json": {
            "data": {
              "format": "json"
            }
          }
        }
      }
    }
  }
}
EOF

```


### graylog
    
```
(((EventID:"7" AND ImageLoaded.keyword:(*\\dbghelp.dll *\\dbgcore.dll) AND Image.keyword:(*\\msbuild.exe *\\cmd.exe *\\svchost.exe *\\rundll32.exe *\\powershell.exe *\\word.exe *\\excel.exe *\\powerpnt.exe *\\outlook.exe *\\monitoringhost.exe *\\wmic.exe *\\msiexec.exe *\\bash.exe *\\wscript.exe *\\cscript.exe *\\mshta.exe *\\regsvr32.exe *\\schtasks.exe *\\dnx.exe *\\regsvcs.exe *\\sc.exe *\\scriptrunner.exe)) AND (NOT (Image.keyword:*Visual Studio*))) OR ((EventID:"7" AND ImageLoaded.keyword:(*\\dbghelp.dll *\\dbgcore.dll) AND Signed:"FALSE") AND (NOT (Image.keyword:*Visual Studio*))))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" (((EventCode="7" (ImageLoaded="*\\dbghelp.dll" OR ImageLoaded="*\\dbgcore.dll") (Image="*\\msbuild.exe" OR Image="*\\cmd.exe" OR Image="*\\svchost.exe" OR Image="*\\rundll32.exe" OR Image="*\\powershell.exe" OR Image="*\\word.exe" OR Image="*\\excel.exe" OR Image="*\\powerpnt.exe" OR Image="*\\outlook.exe" OR Image="*\\monitoringhost.exe" OR Image="*\\wmic.exe" OR Image="*\\msiexec.exe" OR Image="*\\bash.exe" OR Image="*\\wscript.exe" OR Image="*\\cscript.exe" OR Image="*\\mshta.exe" OR Image="*\\regsvr32.exe" OR Image="*\\schtasks.exe" OR Image="*\\dnx.exe" OR Image="*\\regsvcs.exe" OR Image="*\\sc.exe" OR Image="*\\scriptrunner.exe")) NOT (Image="*Visual Studio*")) OR ((EventCode="7" (ImageLoaded="*\\dbghelp.dll" OR ImageLoaded="*\\dbgcore.dll") Signed="FALSE") NOT (Image="*Visual Studio*")))) | table ComputerName,User,Image,ImageLoaded
```


### logpoint
    
```
(((event_id="7" ImageLoaded IN ["*\\dbghelp.dll", "*\\dbgcore.dll"] Image IN ["*\\msbuild.exe", "*\\cmd.exe", "*\\svchost.exe", "*\\rundll32.exe", "*\\powershell.exe", "*\\word.exe", "*\\excel.exe", "*\\powerpnt.exe", "*\\outlook.exe", "*\\monitoringhost.exe", "*\\wmic.exe", "*\\msiexec.exe", "*\\bash.exe", "*\\wscript.exe", "*\\cscript.exe", "*\\mshta.exe", "*\\regsvr32.exe", "*\\schtasks.exe", "*\\dnx.exe", "*\\regsvcs.exe", "*\\sc.exe", "*\\scriptrunner.exe"])  -(Image="*Visual Studio*")) OR ((event_id="7" ImageLoaded IN ["*\\dbghelp.dll", "*\\dbgcore.dll"] Signed="FALSE")  -(Image="*Visual Studio*")))
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?=.*(?:.*(?=.*7)(?=.*(?:.*.*\dbghelp\.dll|.*.*\dbgcore\.dll))(?=.*(?:.*.*\msbuild\.exe|.*.*\cmd\.exe|.*.*\svchost\.exe|.*.*\rundll32\.exe|.*.*\powershell\.exe|.*.*\word\.exe|.*.*\excel\.exe|.*.*\powerpnt\.exe|.*.*\outlook\.exe|.*.*\monitoringhost\.exe|.*.*\wmic\.exe|.*.*\msiexec\.exe|.*.*\bash\.exe|.*.*\wscript\.exe|.*.*\cscript\.exe|.*.*\mshta\.exe|.*.*\regsvr32\.exe|.*.*\schtasks\.exe|.*.*\dnx\.exe|.*.*\regsvcs\.exe|.*.*\sc\.exe|.*.*\scriptrunner\.exe))))(?=.*(?!.*(?:.*(?=.*.*Visual Studio.*)))))|.*(?:.*(?=.*(?:.*(?=.*7)(?=.*(?:.*.*\dbghelp\.dll|.*.*\dbgcore\.dll))(?=.*FALSE)))(?=.*(?!.*(?:.*(?=.*.*Visual Studio.*)))))))'
```



