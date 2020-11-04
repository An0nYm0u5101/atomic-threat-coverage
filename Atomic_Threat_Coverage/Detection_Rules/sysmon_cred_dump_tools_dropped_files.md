| Title                    | Cred Dump Tools Dropped Files       |
|:-------------------------|:------------------|
| **Description**          | Files with well-known filenames (parts of credential dump software or files produced by them) creation |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0015_11_windows_sysmon_FileCreate](../Data_Needed/DN_0015_11_windows_sysmon_FileCreate.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Legitimate Administrator using tool for password recovery</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment](https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment)</li></ul>  |
| **Author**               | Teymur Kheirkhabarov, oscd.community |


## Detection Rules

### Sigma rule

```
title: Cred Dump Tools Dropped Files
id: 8fbf3271-1ef6-4e94-8210-03c2317947f6
description: Files with well-known filenames (parts of credential dump software or files produced by them) creation
author: Teymur Kheirkhabarov, oscd.community
date: 2019/11/01
modified: 2019/11/13
references:
    - https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment
tags:
    - attack.credential_access
    - attack.t1003
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 11
        TargetFilename|contains: 
            - '\pwdump'
            - '\kirbi'
            - '\pwhashes'
            - '\wce_ccache'
            - '\wce_krbtkts'
            - '\fgdump-log'
        TargetFilename|endswith: 
            - '\test.pwd'
            - '\lsremora64.dll'
            - '\lsremora.dll'
            - '\fgexec.exe'
            - '\wceaux.dll'
            - '\SAM.out'
            - '\SECURITY.out'
            - '\SYSTEM.out'
            - '\NTDS.out'
            - '\DumpExt.dll'
            - '\DumpSvc.exe'
            - '\cachedump64.exe'
            - '\cachedump.exe'
            - '\pstgdump.exe'
            - '\servpw.exe'
            - '\servpw64.exe'
            - '\pwdump.exe'
            - '\procdump64.exe'
    condition: selection
falsepositives:
    - Legitimate Administrator using tool for password recovery
level: high
status: experimental

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "11" -and ($_.message -match "TargetFilename.*.*\\pwdump.*" -or $_.message -match "TargetFilename.*.*\\kirbi.*" -or $_.message -match "TargetFilename.*.*\\pwhashes.*" -or $_.message -match "TargetFilename.*.*\\wce_ccache.*" -or $_.message -match "TargetFilename.*.*\\wce_krbtkts.*" -or $_.message -match "TargetFilename.*.*\\fgdump-log.*") -and ($_.message -match "TargetFilename.*.*\\test.pwd" -or $_.message -match "TargetFilename.*.*\\lsremora64.dll" -or $_.message -match "TargetFilename.*.*\\lsremora.dll" -or $_.message -match "TargetFilename.*.*\\fgexec.exe" -or $_.message -match "TargetFilename.*.*\\wceaux.dll" -or $_.message -match "TargetFilename.*.*\\SAM.out" -or $_.message -match "TargetFilename.*.*\\SECURITY.out" -or $_.message -match "TargetFilename.*.*\\SYSTEM.out" -or $_.message -match "TargetFilename.*.*\\NTDS.out" -or $_.message -match "TargetFilename.*.*\\DumpExt.dll" -or $_.message -match "TargetFilename.*.*\\DumpSvc.exe" -or $_.message -match "TargetFilename.*.*\\cachedump64.exe" -or $_.message -match "TargetFilename.*.*\\cachedump.exe" -or $_.message -match "TargetFilename.*.*\\pstgdump.exe" -or $_.message -match "TargetFilename.*.*\\servpw.exe" -or $_.message -match "TargetFilename.*.*\\servpw64.exe" -or $_.message -match "TargetFilename.*.*\\pwdump.exe" -or $_.message -match "TargetFilename.*.*\\procdump64.exe")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND winlog.event_id:"11" AND winlog.event_data.TargetFilename.keyword:(*\\pwdump* OR *\\kirbi* OR *\\pwhashes* OR *\\wce_ccache* OR *\\wce_krbtkts* OR *\\fgdump\-log*) AND winlog.event_data.TargetFilename.keyword:(*\\test.pwd OR *\\lsremora64.dll OR *\\lsremora.dll OR *\\fgexec.exe OR *\\wceaux.dll OR *\\SAM.out OR *\\SECURITY.out OR *\\SYSTEM.out OR *\\NTDS.out OR *\\DumpExt.dll OR *\\DumpSvc.exe OR *\\cachedump64.exe OR *\\cachedump.exe OR *\\pstgdump.exe OR *\\servpw.exe OR *\\servpw64.exe OR *\\pwdump.exe OR *\\procdump64.exe))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/8fbf3271-1ef6-4e94-8210-03c2317947f6 <<EOF
{
  "metadata": {
    "title": "Cred Dump Tools Dropped Files",
    "description": "Files with well-known filenames (parts of credential dump software or files produced by them) creation",
    "tags": [
      "attack.credential_access",
      "attack.t1003"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"11\" AND winlog.event_data.TargetFilename.keyword:(*\\\\pwdump* OR *\\\\kirbi* OR *\\\\pwhashes* OR *\\\\wce_ccache* OR *\\\\wce_krbtkts* OR *\\\\fgdump\\-log*) AND winlog.event_data.TargetFilename.keyword:(*\\\\test.pwd OR *\\\\lsremora64.dll OR *\\\\lsremora.dll OR *\\\\fgexec.exe OR *\\\\wceaux.dll OR *\\\\SAM.out OR *\\\\SECURITY.out OR *\\\\SYSTEM.out OR *\\\\NTDS.out OR *\\\\DumpExt.dll OR *\\\\DumpSvc.exe OR *\\\\cachedump64.exe OR *\\\\cachedump.exe OR *\\\\pstgdump.exe OR *\\\\servpw.exe OR *\\\\servpw64.exe OR *\\\\pwdump.exe OR *\\\\procdump64.exe))"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"11\" AND winlog.event_data.TargetFilename.keyword:(*\\\\pwdump* OR *\\\\kirbi* OR *\\\\pwhashes* OR *\\\\wce_ccache* OR *\\\\wce_krbtkts* OR *\\\\fgdump\\-log*) AND winlog.event_data.TargetFilename.keyword:(*\\\\test.pwd OR *\\\\lsremora64.dll OR *\\\\lsremora.dll OR *\\\\fgexec.exe OR *\\\\wceaux.dll OR *\\\\SAM.out OR *\\\\SECURITY.out OR *\\\\SYSTEM.out OR *\\\\NTDS.out OR *\\\\DumpExt.dll OR *\\\\DumpSvc.exe OR *\\\\cachedump64.exe OR *\\\\cachedump.exe OR *\\\\pstgdump.exe OR *\\\\servpw.exe OR *\\\\servpw64.exe OR *\\\\pwdump.exe OR *\\\\procdump64.exe))",
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
        "subject": "Sigma Rule 'Cred Dump Tools Dropped Files'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}{{_source}}\n================================================================================\n{{/ctx.payload.hits.hits}}",
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
(EventID:"11" AND TargetFilename.keyword:(*\\pwdump* *\\kirbi* *\\pwhashes* *\\wce_ccache* *\\wce_krbtkts* *\\fgdump\-log*) AND TargetFilename.keyword:(*\\test.pwd *\\lsremora64.dll *\\lsremora.dll *\\fgexec.exe *\\wceaux.dll *\\SAM.out *\\SECURITY.out *\\SYSTEM.out *\\NTDS.out *\\DumpExt.dll *\\DumpSvc.exe *\\cachedump64.exe *\\cachedump.exe *\\pstgdump.exe *\\servpw.exe *\\servpw64.exe *\\pwdump.exe *\\procdump64.exe))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="11" (TargetFilename="*\\pwdump*" OR TargetFilename="*\\kirbi*" OR TargetFilename="*\\pwhashes*" OR TargetFilename="*\\wce_ccache*" OR TargetFilename="*\\wce_krbtkts*" OR TargetFilename="*\\fgdump-log*") (TargetFilename="*\\test.pwd" OR TargetFilename="*\\lsremora64.dll" OR TargetFilename="*\\lsremora.dll" OR TargetFilename="*\\fgexec.exe" OR TargetFilename="*\\wceaux.dll" OR TargetFilename="*\\SAM.out" OR TargetFilename="*\\SECURITY.out" OR TargetFilename="*\\SYSTEM.out" OR TargetFilename="*\\NTDS.out" OR TargetFilename="*\\DumpExt.dll" OR TargetFilename="*\\DumpSvc.exe" OR TargetFilename="*\\cachedump64.exe" OR TargetFilename="*\\cachedump.exe" OR TargetFilename="*\\pstgdump.exe" OR TargetFilename="*\\servpw.exe" OR TargetFilename="*\\servpw64.exe" OR TargetFilename="*\\pwdump.exe" OR TargetFilename="*\\procdump64.exe"))
```


### logpoint
    
```
(event_id="11" TargetFilename IN ["*\\pwdump*", "*\\kirbi*", "*\\pwhashes*", "*\\wce_ccache*", "*\\wce_krbtkts*", "*\\fgdump-log*"] TargetFilename IN ["*\\test.pwd", "*\\lsremora64.dll", "*\\lsremora.dll", "*\\fgexec.exe", "*\\wceaux.dll", "*\\SAM.out", "*\\SECURITY.out", "*\\SYSTEM.out", "*\\NTDS.out", "*\\DumpExt.dll", "*\\DumpSvc.exe", "*\\cachedump64.exe", "*\\cachedump.exe", "*\\pstgdump.exe", "*\\servpw.exe", "*\\servpw64.exe", "*\\pwdump.exe", "*\\procdump64.exe"])
```


### grep
    
```
grep -P '^(?:.*(?=.*11)(?=.*(?:.*.*\pwdump.*|.*.*\kirbi.*|.*.*\pwhashes.*|.*.*\wce_ccache.*|.*.*\wce_krbtkts.*|.*.*\fgdump-log.*))(?=.*(?:.*.*\test\.pwd|.*.*\lsremora64\.dll|.*.*\lsremora\.dll|.*.*\fgexec\.exe|.*.*\wceaux\.dll|.*.*\SAM\.out|.*.*\SECURITY\.out|.*.*\SYSTEM\.out|.*.*\NTDS\.out|.*.*\DumpExt\.dll|.*.*\DumpSvc\.exe|.*.*\cachedump64\.exe|.*.*\cachedump\.exe|.*.*\pstgdump\.exe|.*.*\servpw\.exe|.*.*\servpw64\.exe|.*.*\pwdump\.exe|.*.*\procdump64\.exe)))'
```



