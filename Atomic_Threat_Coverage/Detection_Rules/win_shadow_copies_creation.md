| Title                    | Shadow Copies Creation Using Operating Systems Utilities       |
|:-------------------------|:------------------|
| **Description**          | Shadow Copies creation using operating systems utilities, possible credential access |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Legitimate administrator working with shadow copies, access for backup purposes</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment](https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment)</li><li>[https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/tutorial-for-ntds-goodness-vssadmin-wmis-ntdsdit-system/](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/tutorial-for-ntds-goodness-vssadmin-wmis-ntdsdit-system/)</li></ul>  |
| **Author**               | Teymur Kheirkhabarov, Daniil Yugoslavskiy, oscd.community |


## Detection Rules

### Sigma rule

```
title: Shadow Copies Creation Using Operating Systems Utilities
id: b17ea6f7-6e90-447e-a799-e6c0a493d6ce
description: Shadow Copies creation using operating systems utilities, possible credential access
author: Teymur Kheirkhabarov, Daniil Yugoslavskiy, oscd.community
date: 2019/10/22
references:
    - https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment
    - https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/tutorial-for-ntds-goodness-vssadmin-wmis-ntdsdit-system/
tags:
    - attack.credential_access
    - attack.t1003
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        NewProcessName|endswith:
            - '\powershell.exe'
            - '\wmic.exe'
            - '\vssadmin.exe'
        CommandLine|contains|all:
            - shadow
            - create
    condition: selection
falsepositives:
    - Legitimate administrator working with shadow copies, access for backup purposes
status: experimental
level: medium

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "NewProcessName.*.*\\powershell.exe" -or $_.message -match "NewProcessName.*.*\\wmic.exe" -or $_.message -match "NewProcessName.*.*\\vssadmin.exe") -and $_.message -match "CommandLine.*.*shadow.*" -and $_.message -match "CommandLine.*.*create.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.NewProcessName.keyword:(*\\powershell.exe OR *\\wmic.exe OR *\\vssadmin.exe) AND winlog.event_data.CommandLine.keyword:*shadow* AND winlog.event_data.CommandLine.keyword:*create*)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/b17ea6f7-6e90-447e-a799-e6c0a493d6ce <<EOF
{
  "metadata": {
    "title": "Shadow Copies Creation Using Operating Systems Utilities",
    "description": "Shadow Copies creation using operating systems utilities, possible credential access",
    "tags": [
      "attack.credential_access",
      "attack.t1003"
    ],
    "query": "(winlog.event_data.NewProcessName.keyword:(*\\\\powershell.exe OR *\\\\wmic.exe OR *\\\\vssadmin.exe) AND winlog.event_data.CommandLine.keyword:*shadow* AND winlog.event_data.CommandLine.keyword:*create*)"
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
                    "query": "(winlog.event_data.NewProcessName.keyword:(*\\\\powershell.exe OR *\\\\wmic.exe OR *\\\\vssadmin.exe) AND winlog.event_data.CommandLine.keyword:*shadow* AND winlog.event_data.CommandLine.keyword:*create*)",
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
        "subject": "Sigma Rule 'Shadow Copies Creation Using Operating Systems Utilities'",
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
(NewProcessName.keyword:(*\\powershell.exe *\\wmic.exe *\\vssadmin.exe) AND CommandLine.keyword:*shadow* AND CommandLine.keyword:*create*)
```


### splunk
    
```
((NewProcessName="*\\powershell.exe" OR NewProcessName="*\\wmic.exe" OR NewProcessName="*\\vssadmin.exe") CommandLine="*shadow*" CommandLine="*create*")
```


### logpoint
    
```
(NewProcessName IN ["*\\powershell.exe", "*\\wmic.exe", "*\\vssadmin.exe"] CommandLine="*shadow*" CommandLine="*create*")
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*.*\powershell\.exe|.*.*\wmic\.exe|.*.*\vssadmin\.exe))(?=.*.*shadow.*)(?=.*.*create.*))'
```



