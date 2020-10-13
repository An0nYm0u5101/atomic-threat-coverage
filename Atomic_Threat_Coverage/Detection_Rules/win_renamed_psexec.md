| Title                    | Renamed PsExec       |
|:-------------------------|:------------------|
| **Description**          | Detects the execution of a renamed PsExec often used by attackers or malware |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1036: Masquerading](https://attack.mitre.org/techniques/T1036)</li><li>[T1036.003: Rename System Utilities](https://attack.mitre.org/techniques/T1036/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1036.003: Rename System Utilities](../Triggers/T1036.003.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Software that illegaly integrates PsExec in a renamed form</li><li>Administrators that have renamed PsExec and no one knows why</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.trendmicro.com/vinfo/hk-en/security/news/cybercrime-and-digital-threats/megacortex-ransomware-spotted-attacking-enterprise-networks](https://www.trendmicro.com/vinfo/hk-en/security/news/cybercrime-and-digital-threats/megacortex-ransomware-spotted-attacking-enterprise-networks)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>car.2013-05-009</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Renamed PsExec
id: a7a7e0e5-1d57-49df-9c58-9fe5bc0346a2
status: experimental
description: Detects the execution of a renamed PsExec often used by attackers or malware
references:
    - https://www.trendmicro.com/vinfo/hk-en/security/news/cybercrime-and-digital-threats/megacortex-ransomware-spotted-attacking-enterprise-networks
author: Florian Roth
date: 2019/05/21
modified: 2020/09/06
tags:
    - car.2013-05-009
    - attack.defense_evasion
    - attack.t1036 # an old one
    - attack.t1036.003
logsource:
    product: windows
    category: process_creation
detection:
    selection:
        Description: 'Execute processes remotely'
        Product: 'Sysinternals PsExec'
    filter:
        Image:
           - '*\PsExec.exe'
           - '*\PsExec64.exe'
    condition: selection and not filter
falsepositives:
    - Software that illegaly integrates PsExec in a renamed form
    - Administrators that have renamed PsExec and no one knows why
level: high

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "Description.*Execute processes remotely" -and $_.message -match "Product.*Sysinternals PsExec") -and  -not (($_.message -match "Image.*.*\\PsExec.exe" -or $_.message -match "Image.*.*\\PsExec64.exe"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Description:"Execute\ processes\ remotely" AND Product:"Sysinternals\ PsExec") AND (NOT (winlog.event_data.Image.keyword:(*\\PsExec.exe OR *\\PsExec64.exe))))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/a7a7e0e5-1d57-49df-9c58-9fe5bc0346a2 <<EOF
{
  "metadata": {
    "title": "Renamed PsExec",
    "description": "Detects the execution of a renamed PsExec often used by attackers or malware",
    "tags": [
      "car.2013-05-009",
      "attack.defense_evasion",
      "attack.t1036",
      "attack.t1036.003"
    ],
    "query": "((winlog.event_data.Description:\"Execute\\ processes\\ remotely\" AND Product:\"Sysinternals\\ PsExec\") AND (NOT (winlog.event_data.Image.keyword:(*\\\\PsExec.exe OR *\\\\PsExec64.exe))))"
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
                    "query": "((winlog.event_data.Description:\"Execute\\ processes\\ remotely\" AND Product:\"Sysinternals\\ PsExec\") AND (NOT (winlog.event_data.Image.keyword:(*\\\\PsExec.exe OR *\\\\PsExec64.exe))))",
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
      "throttle_period": "15m",
      "email": {
        "profile": "standard",
        "from": "root@localhost",
        "to": "root@localhost",
        "subject": "Sigma Rule 'Renamed PsExec'",
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
((Description:"Execute processes remotely" AND Product:"Sysinternals PsExec") AND (NOT (Image.keyword:(*\\PsExec.exe *\\PsExec64.exe))))
```


### splunk
    
```
((Description="Execute processes remotely" Product="Sysinternals PsExec") NOT ((Image="*\\PsExec.exe" OR Image="*\\PsExec64.exe")))
```


### logpoint
    
```
((Description="Execute processes remotely" Product="Sysinternals PsExec")  -(Image IN ["*\\PsExec.exe", "*\\PsExec64.exe"]))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*Execute processes remotely)(?=.*Sysinternals PsExec)))(?=.*(?!.*(?:.*(?=.*(?:.*.*\PsExec\.exe|.*.*\PsExec64\.exe))))))'
```



