| Title                    | Suspicious Csc.exe Source File Folder       |
|:-------------------------|:------------------|
| **Description**          | Detects a suspicious execution of csc.exe, which uses a source in a suspicious folder (e.g. AppData) |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1500: Compile After Delivery](https://attack.mitre.org/techniques/T1500)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1500: Compile After Delivery](../Triggers/T1500.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>https://twitter.com/gN3mes1s/status/1206874118282448897</li><li>https://twitter.com/gabriele_pippi/status/1206907900268072962</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://securityboulevard.com/2019/08/agent-tesla-evading-edr-by-removing-api-hooks/](https://securityboulevard.com/2019/08/agent-tesla-evading-edr-by-removing-api-hooks/)</li><li>[https://www.clearskysec.com/wp-content/uploads/2018/11/MuddyWater-Operations-in-Lebanon-and-Oman.pdf](https://www.clearskysec.com/wp-content/uploads/2018/11/MuddyWater-Operations-in-Lebanon-and-Oman.pdf)</li><li>[https://app.any.run/tasks/c6993447-d1d8-414e-b856-675325e5aa09/](https://app.any.run/tasks/c6993447-d1d8-414e-b856-675325e5aa09/)</li><li>[https://twitter.com/gN3mes1s/status/1206874118282448897](https://twitter.com/gN3mes1s/status/1206874118282448897)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Suspicious Csc.exe Source File Folder
id: dcaa3f04-70c3-427a-80b4-b870d73c94c4
description: Detects a suspicious execution of csc.exe, which uses a source in a suspicious folder (e.g. AppData)
status: experimental
references:
    - https://securityboulevard.com/2019/08/agent-tesla-evading-edr-by-removing-api-hooks/
    - https://www.clearskysec.com/wp-content/uploads/2018/11/MuddyWater-Operations-in-Lebanon-and-Oman.pdf
    - https://app.any.run/tasks/c6993447-d1d8-414e-b856-675325e5aa09/
    - https://twitter.com/gN3mes1s/status/1206874118282448897
author: Florian Roth
date: 2019/08/24
modified: 2019/12/17
tags:
    - attack.defense_evasion
    - attack.t1500
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image: '*\csc.exe'
        CommandLine: 
            - '*\AppData\\*'
            - '*\Windows\Temp\\*'
    filter:
        ParentImage: 
            - 'C:\Program Files*'  # https://twitter.com/gN3mes1s/status/1206874118282448897
            - '*\sdiagnhost.exe'  # https://twitter.com/gN3mes1s/status/1206874118282448897
            - '*\w3wp.exe'  # https://twitter.com/gabriele_pippi/status/1206907900268072962
    condition: selection and not filter
falsepositives:
    - https://twitter.com/gN3mes1s/status/1206874118282448897
    - https://twitter.com/gabriele_pippi/status/1206907900268072962
level: high

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "Image.*.*\\csc.exe" -and ($_.message -match "CommandLine.*.*\\AppData\\.*" -or $_.message -match "CommandLine.*.*\\Windows\\Temp\\.*")) -and  -not (($_.message -match "ParentImage.*C:\\Program Files.*" -or $_.message -match "ParentImage.*.*\\sdiagnhost.exe" -or $_.message -match "ParentImage.*.*\\w3wp.exe"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Image.keyword:*\\csc.exe AND winlog.event_data.CommandLine.keyword:(*\\AppData\\* OR *\\Windows\\Temp\\*)) AND (NOT (winlog.event_data.ParentImage.keyword:(C\:\\Program\ Files* OR *\\sdiagnhost.exe OR *\\w3wp.exe))))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/dcaa3f04-70c3-427a-80b4-b870d73c94c4 <<EOF
{
  "metadata": {
    "title": "Suspicious Csc.exe Source File Folder",
    "description": "Detects a suspicious execution of csc.exe, which uses a source in a suspicious folder (e.g. AppData)",
    "tags": [
      "attack.defense_evasion",
      "attack.t1500"
    ],
    "query": "((winlog.event_data.Image.keyword:*\\\\csc.exe AND winlog.event_data.CommandLine.keyword:(*\\\\AppData\\\\* OR *\\\\Windows\\\\Temp\\\\*)) AND (NOT (winlog.event_data.ParentImage.keyword:(C\\:\\\\Program\\ Files* OR *\\\\sdiagnhost.exe OR *\\\\w3wp.exe))))"
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
                    "query": "((winlog.event_data.Image.keyword:*\\\\csc.exe AND winlog.event_data.CommandLine.keyword:(*\\\\AppData\\\\* OR *\\\\Windows\\\\Temp\\\\*)) AND (NOT (winlog.event_data.ParentImage.keyword:(C\\:\\\\Program\\ Files* OR *\\\\sdiagnhost.exe OR *\\\\w3wp.exe))))",
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
        "subject": "Sigma Rule 'Suspicious Csc.exe Source File Folder'",
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
((Image.keyword:*\\csc.exe AND CommandLine.keyword:(*\\AppData\\* *\\Windows\\Temp\\*)) AND (NOT (ParentImage.keyword:(C\:\\Program Files* *\\sdiagnhost.exe *\\w3wp.exe))))
```


### splunk
    
```
((Image="*\\csc.exe" (CommandLine="*\\AppData\\*" OR CommandLine="*\\Windows\\Temp\\*")) NOT ((ParentImage="C:\\Program Files*" OR ParentImage="*\\sdiagnhost.exe" OR ParentImage="*\\w3wp.exe")))
```


### logpoint
    
```
((Image="*\\csc.exe" CommandLine IN ["*\\AppData\\*", "*\\Windows\\Temp\\*"])  -(ParentImage IN ["C:\\Program Files*", "*\\sdiagnhost.exe", "*\\w3wp.exe"]))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*.*\csc\.exe)(?=.*(?:.*.*\AppData\\.*|.*.*\Windows\Temp\\.*))))(?=.*(?!.*(?:.*(?=.*(?:.*C:\Program Files.*|.*.*\sdiagnhost\.exe|.*.*\w3wp\.exe))))))'
```



