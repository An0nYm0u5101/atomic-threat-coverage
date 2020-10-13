| Title                    | NTFS Alternate Data Stream       |
|:-------------------------|:------------------|
| **Description**          | Detects writing data into NTFS alternate data streams from powershell. Needs Script Block Logging. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1564.004: NTFS File Attributes](https://attack.mitre.org/techniques/T1564/004)</li><li>[T1096: NTFS File Attributes](https://attack.mitre.org/techniques/T1096)</li><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0036_4104_windows_powershell_script_block](../Data_Needed/DN_0036_4104_windows_powershell_script_block.md)</li><li>[DN_0037_4103_windows_powershell_executing_pipeline](../Data_Needed/DN_0037_4103_windows_powershell_executing_pipeline.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1564.004: NTFS File Attributes](../Triggers/T1564.004.md)</li><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[http://www.powertheshell.com/ntfsstreams/](http://www.powertheshell.com/ntfsstreams/)</li></ul>  |
| **Author**               | Sami Ruohonen |


## Detection Rules

### Sigma rule

```
title: NTFS Alternate Data Stream
id: 8c521530-5169-495d-a199-0a3a881ad24e
status: experimental
description: Detects writing data into NTFS alternate data streams from powershell. Needs Script Block Logging.
references:
    - http://www.powertheshell.com/ntfsstreams/
tags:
    - attack.defense_evasion
    - attack.t1564.004
    - attack.t1096  # an old one
    - attack.execution
    - attack.t1059.001
    - attack.t1086  # an old one
author: Sami Ruohonen
date: 2018/07/24
modified: 2020/08/24
logsource:
    product: windows
    service: powershell
    definition: 'It is recommended to use the new "Script Block Logging" of PowerShell v5 https://adsecurity.org/?p=2277'
detection:
    keyword1:
        - "set-content"
        - "add-content"
    keyword2:
        - "-stream"
    condition: keyword1 and keyword2
falsepositives:
    - unknown
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-PowerShell/Operational | where {(($_.message -match "set-content" -or $_.message -match "add-content") -and $_.message -match "-stream") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(\*.keyword:(*set\-content* OR *add\-content*) AND "\-stream")
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/8c521530-5169-495d-a199-0a3a881ad24e <<EOF
{
  "metadata": {
    "title": "NTFS Alternate Data Stream",
    "description": "Detects writing data into NTFS alternate data streams from powershell. Needs Script Block Logging.",
    "tags": [
      "attack.defense_evasion",
      "attack.t1564.004",
      "attack.t1096",
      "attack.execution",
      "attack.t1059.001",
      "attack.t1086"
    ],
    "query": "(\\*.keyword:(*set\\-content* OR *add\\-content*) AND \"\\-stream\")"
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
                    "query": "(\\*.keyword:(*set\\-content* OR *add\\-content*) AND \"\\-stream\")",
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
        "subject": "Sigma Rule 'NTFS Alternate Data Stream'",
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
(\*.keyword:(*set\-content* OR *add\-content*) AND "\-stream")
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-PowerShell/Operational" ("set-content" OR "add-content") "-stream")
```


### logpoint
    
```
(("set-content" OR "add-content") "-stream")
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?:.*set-content|.*add-content)))(?=.*-stream))'
```



