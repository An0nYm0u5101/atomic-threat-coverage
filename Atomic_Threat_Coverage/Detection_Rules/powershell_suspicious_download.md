| Title                    | Suspicious PowerShell Download       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious PowerShell download command |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li></ul>  |
| **Data Needed**          |  There is no documented Data Needed for this Detection Rule yet  |
| **Trigger**              | <ul><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>PowerShell scripts that download content from the Internet</li></ul>  |
| **Development Status**   | experimental |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Suspicious PowerShell Download
id: 65531a81-a694-4e31-ae04-f8ba5bc33759
status: experimental
description: Detects suspicious PowerShell download command
tags:
    - attack.execution
    - attack.t1059.001
    - attack.t1086  #an old one
author: Florian Roth
date: 2017/03/05
logsource:
    product: windows
    service: powershell
detection:
    downloadfile:
        Message|contains|all:
            - 'System.Net.WebClient'
            - '.DownloadFile('
    downloadstring:
        Message|contains|all:
            - 'System.Net.WebClient'
            - '.DownloadString('
    condition: downloadfile or downloadstring
falsepositives:
    - PowerShell scripts that download content from the Internet
level: medium

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-PowerShell/Operational | where {($_.message -match ".*System.Net.WebClient.*" -and ($_.message -match ".*.DownloadFile(.*" -or $_.message -match ".*.DownloadString(.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(Message.keyword:*System.Net.WebClient* AND (Message.keyword:*.DownloadFile\(* OR Message.keyword:*.DownloadString\(*))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/65531a81-a694-4e31-ae04-f8ba5bc33759 <<EOF
{
  "metadata": {
    "title": "Suspicious PowerShell Download",
    "description": "Detects suspicious PowerShell download command",
    "tags": [
      "attack.execution",
      "attack.t1059.001",
      "attack.t1086"
    ],
    "query": "(Message.keyword:*System.Net.WebClient* AND (Message.keyword:*.DownloadFile\\(* OR Message.keyword:*.DownloadString\\(*))"
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
                    "query": "(Message.keyword:*System.Net.WebClient* AND (Message.keyword:*.DownloadFile\\(* OR Message.keyword:*.DownloadString\\(*))",
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
        "subject": "Sigma Rule 'Suspicious PowerShell Download'",
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
(Message.keyword:*System.Net.WebClient* AND (Message.keyword:*.DownloadFile\(* OR Message.keyword:*.DownloadString\(*))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-PowerShell/Operational" Message="*System.Net.WebClient*" (Message="*.DownloadFile(*" OR Message="*.DownloadString(*"))
```


### logpoint
    
```
(Message="*System.Net.WebClient*" (Message="*.DownloadFile(*" OR Message="*.DownloadString(*"))
```


### grep
    
```
grep -P '^(?:.*(?=.*.*System\.Net\.WebClient.*)(?=.*(?:.*(?:.*.*\.DownloadFile\(.*|.*.*\.DownloadString\(.*))))'
```



