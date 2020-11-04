| Title                    | Svchost DLL Search Order Hijack       |
|:-------------------------|:------------------|
| **Description**          | IKEEXT and SessionEnv service, as they call LoadLibrary on files that do not exist within C:\Windows\System32\ by default. An attacker can place their malicious logic within the PROCESS_ATTACH block of their library and restart the aforementioned services "svchost.exe -k netsvcs" to gain code execution on a remote machine. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1073: DLL Side-Loading](https://attack.mitre.org/techniques/T1073)</li><li>[T1038: DLL Search Order Hijacking](https://attack.mitre.org/techniques/T1038)</li><li>[T1112: Modify Registry](https://attack.mitre.org/techniques/T1112)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0011_7_windows_sysmon_image_loaded](../Data_Needed/DN_0011_7_windows_sysmon_image_loaded.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1073: DLL Side-Loading](../Triggers/T1073.md)</li><li>[T1038: DLL Search Order Hijacking](../Triggers/T1038.md)</li><li>[T1112: Modify Registry](../Triggers/T1112.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Pentest</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://posts.specterops.io/lateral-movement-scm-and-dll-hijacking-primer-d2f61e8ab992](https://posts.specterops.io/lateral-movement-scm-and-dll-hijacking-primer-d2f61e8ab992)</li></ul>  |
| **Author**               | SBousseaden |


## Detection Rules

### Sigma rule

```
title: Svchost DLL Search Order Hijack
id: 602a1f13-c640-4d73-b053-be9a2fa58b77
status: experimental
description: IKEEXT and SessionEnv service, as they call LoadLibrary on files that do not exist within C:\Windows\System32\ by default. An attacker can place their
    malicious logic within the PROCESS_ATTACH block of their library and restart the aforementioned services "svchost.exe -k netsvcs" to gain code execution on a
    remote machine.
references:
    - https://posts.specterops.io/lateral-movement-scm-and-dll-hijacking-primer-d2f61e8ab992
author: SBousseaden
date: 2019/10/28
tags:
    - attack.persistence
    - attack.defense_evasion
    - attack.t1073
    - attack.t1038
    - attack.t1112
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 7
        Image:
            - '*\svchost.exe'
        ImageLoaded:
            - '*\tsmsisrv.dll'
            - '*\tsvipsrv.dll'
            - '*\wlbsctrl.dll'
    filter:
        EventID: 7
        Image:
            - '*\svchost.exe'
        ImageLoaded:
            - 'C:\Windows\WinSxS\*'        
    condition: selection and not filter
falsepositives:
    - Pentest
level: high
```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {(($_.ID -eq "7" -and ($_.message -match "Image.*.*\\svchost.exe") -and ($_.message -match "ImageLoaded.*.*\\tsmsisrv.dll" -or $_.message -match "ImageLoaded.*.*\\tsvipsrv.dll" -or $_.message -match "ImageLoaded.*.*\\wlbsctrl.dll")) -and  -not ($_.ID -eq "7" -and ($_.message -match "Image.*.*\\svchost.exe") -and ($_.message -match "ImageLoaded.*C:\\Windows\\WinSxS\\.*"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND (winlog.event_id:"7" AND winlog.event_data.Image.keyword:(*\\svchost.exe) AND winlog.event_data.ImageLoaded.keyword:(*\\tsmsisrv.dll OR *\\tsvipsrv.dll OR *\\wlbsctrl.dll)) AND (NOT (winlog.event_id:"7" AND winlog.event_data.Image.keyword:(*\\svchost.exe) AND winlog.event_data.ImageLoaded:("C\:\\Windows\\WinSxS\*"))))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/602a1f13-c640-4d73-b053-be9a2fa58b77 <<EOF
{
  "metadata": {
    "title": "Svchost DLL Search Order Hijack",
    "description": "IKEEXT and SessionEnv service, as they call LoadLibrary on files that do not exist within C:\\Windows\\System32\\ by default. An attacker can place their malicious logic within the PROCESS_ATTACH block of their library and restart the aforementioned services \"svchost.exe -k netsvcs\" to gain code execution on a remote machine.",
    "tags": [
      "attack.persistence",
      "attack.defense_evasion",
      "attack.t1073",
      "attack.t1038",
      "attack.t1112"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"7\" AND winlog.event_data.Image.keyword:(*\\\\svchost.exe) AND winlog.event_data.ImageLoaded.keyword:(*\\\\tsmsisrv.dll OR *\\\\tsvipsrv.dll OR *\\\\wlbsctrl.dll)) AND (NOT (winlog.event_id:\"7\" AND winlog.event_data.Image.keyword:(*\\\\svchost.exe) AND winlog.event_data.ImageLoaded:(\"C\\:\\\\Windows\\\\WinSxS\\*\"))))"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"7\" AND winlog.event_data.Image.keyword:(*\\\\svchost.exe) AND winlog.event_data.ImageLoaded.keyword:(*\\\\tsmsisrv.dll OR *\\\\tsvipsrv.dll OR *\\\\wlbsctrl.dll)) AND (NOT (winlog.event_id:\"7\" AND winlog.event_data.Image.keyword:(*\\\\svchost.exe) AND winlog.event_data.ImageLoaded:(\"C\\:\\\\Windows\\\\WinSxS\\*\"))))",
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
        "subject": "Sigma Rule 'Svchost DLL Search Order Hijack'",
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
((EventID:"7" AND Image.keyword:(*\\svchost.exe) AND ImageLoaded.keyword:(*\\tsmsisrv.dll *\\tsvipsrv.dll *\\wlbsctrl.dll)) AND (NOT (EventID:"7" AND Image.keyword:(*\\svchost.exe) AND ImageLoaded:("C\:\\Windows\\WinSxS\*"))))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode="7" (Image="*\\svchost.exe") (ImageLoaded="*\\tsmsisrv.dll" OR ImageLoaded="*\\tsvipsrv.dll" OR ImageLoaded="*\\wlbsctrl.dll")) NOT (EventCode="7" (Image="*\\svchost.exe") (ImageLoaded="C:\\Windows\\WinSxS\*")))
```


### logpoint
    
```
((event_id="7" Image IN ["*\\svchost.exe"] ImageLoaded IN ["*\\tsmsisrv.dll", "*\\tsvipsrv.dll", "*\\wlbsctrl.dll"])  -(event_id="7" Image IN ["*\\svchost.exe"] ImageLoaded IN ["C:\\Windows\\WinSxS\*"]))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*7)(?=.*(?:.*.*\svchost\.exe))(?=.*(?:.*.*\tsmsisrv\.dll|.*.*\tsvipsrv\.dll|.*.*\wlbsctrl\.dll))))(?=.*(?!.*(?:.*(?=.*7)(?=.*(?:.*.*\svchost\.exe))(?=.*(?:.*C:\Windows\WinSxS\.*))))))'
```



