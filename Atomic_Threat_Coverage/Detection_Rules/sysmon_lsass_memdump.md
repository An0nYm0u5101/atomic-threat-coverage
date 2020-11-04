| Title                    | LSASS Memory Dump       |
|:-------------------------|:------------------|
| **Description**          | Detects process LSASS memory dump using procdump or taskmgr based on the CallTrace pointing to dbghelp.dll or dbgcore.dll for win10 |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0014_10_windows_sysmon_ProcessAccess](../Data_Needed/DN_0014_10_windows_sysmon_ProcessAccess.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://blog.menasec.net/2019/02/threat-hunting-21-procdump-or-taskmgr.html](https://blog.menasec.net/2019/02/threat-hunting-21-procdump-or-taskmgr.html)</li></ul>  |
| **Author**               | Samir Bousseaden |
| Other Tags           | <ul><li>attack.s0002</li></ul> | 

## Detection Rules

### Sigma rule

```
title: LSASS Memory Dump
id: 5ef9853e-4d0e-4a70-846f-a9ca37d876da
status: experimental
description: Detects process LSASS memory dump using procdump or taskmgr based on the CallTrace pointing to dbghelp.dll or dbgcore.dll for win10
author: Samir Bousseaden
date: 2019/04/03
references:
    - https://blog.menasec.net/2019/02/threat-hunting-21-procdump-or-taskmgr.html
tags:
    - attack.t1003
    - attack.s0002
    - attack.credential_access
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 10
        TargetImage: 'C:\windows\system32\lsass.exe'
        GrantedAccess: '0x1fffff'
        CallTrace:
         - '*dbghelp.dll*'
         - '*dbgcore.dll*'
    condition: selection
falsepositives:
    - unknown
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "10" -and $_.message -match "TargetImage.*C:\\windows\\system32\\lsass.exe" -and $_.message -match "GrantedAccess.*0x1fffff" -and ($_.message -match "CallTrace.*.*dbghelp.dll.*" -or $_.message -match "CallTrace.*.*dbgcore.dll.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND winlog.event_id:"10" AND winlog.event_data.TargetImage:"C\:\\windows\\system32\\lsass.exe" AND winlog.event_data.GrantedAccess:"0x1fffff" AND winlog.event_data.CallTrace.keyword:(*dbghelp.dll* OR *dbgcore.dll*))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/5ef9853e-4d0e-4a70-846f-a9ca37d876da <<EOF
{
  "metadata": {
    "title": "LSASS Memory Dump",
    "description": "Detects process LSASS memory dump using procdump or taskmgr based on the CallTrace pointing to dbghelp.dll or dbgcore.dll for win10",
    "tags": [
      "attack.t1003",
      "attack.s0002",
      "attack.credential_access"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"10\" AND winlog.event_data.TargetImage:\"C\\:\\\\windows\\\\system32\\\\lsass.exe\" AND winlog.event_data.GrantedAccess:\"0x1fffff\" AND winlog.event_data.CallTrace.keyword:(*dbghelp.dll* OR *dbgcore.dll*))"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"10\" AND winlog.event_data.TargetImage:\"C\\:\\\\windows\\\\system32\\\\lsass.exe\" AND winlog.event_data.GrantedAccess:\"0x1fffff\" AND winlog.event_data.CallTrace.keyword:(*dbghelp.dll* OR *dbgcore.dll*))",
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
        "subject": "Sigma Rule 'LSASS Memory Dump'",
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
(EventID:"10" AND TargetImage:"C\:\\windows\\system32\\lsass.exe" AND GrantedAccess:"0x1fffff" AND CallTrace.keyword:(*dbghelp.dll* *dbgcore.dll*))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="10" TargetImage="C:\\windows\\system32\\lsass.exe" GrantedAccess="0x1fffff" (CallTrace="*dbghelp.dll*" OR CallTrace="*dbgcore.dll*"))
```


### logpoint
    
```
(event_id="10" TargetImage="C:\\windows\\system32\\lsass.exe" GrantedAccess="0x1fffff" CallTrace IN ["*dbghelp.dll*", "*dbgcore.dll*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*10)(?=.*C:\windows\system32\lsass\.exe)(?=.*0x1fffff)(?=.*(?:.*.*dbghelp\.dll.*|.*.*dbgcore\.dll.*)))'
```



