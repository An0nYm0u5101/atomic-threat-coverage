| Title                    | Suspicious Driver Load from Temp       |
|:-------------------------|:------------------|
| **Description**          | Detects a driver load from a temporary directory |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1050: New Service](https://attack.mitre.org/techniques/T1050)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0010_6_windows_sysmon_driver_loaded](../Data_Needed/DN_0010_6_windows_sysmon_driver_loaded.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1050: New Service](../Triggers/T1050.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>there is a relevant set of false positives depending on applications in the environment</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Suspicious Driver Load from Temp
id: 2c4523d5-d481-4ed0-8ec3-7fbf0cb41a75
description: Detects a driver load from a temporary directory
author: Florian Roth
date: 2017/02/12
tags:
    - attack.persistence
    - attack.t1050
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 6
        ImageLoaded: '*\Temp\\*'
    condition: selection
falsepositives:
    - there is a relevant set of false positives depending on applications in the environment
level: medium

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "6" -and $_.message -match "ImageLoaded.*.*\\Temp\\.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND winlog.event_id:"6" AND winlog.event_data.ImageLoaded.keyword:*\\Temp\\*)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/2c4523d5-d481-4ed0-8ec3-7fbf0cb41a75 <<EOF
{
  "metadata": {
    "title": "Suspicious Driver Load from Temp",
    "description": "Detects a driver load from a temporary directory",
    "tags": [
      "attack.persistence",
      "attack.t1050"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"6\" AND winlog.event_data.ImageLoaded.keyword:*\\\\Temp\\\\*)"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"6\" AND winlog.event_data.ImageLoaded.keyword:*\\\\Temp\\\\*)",
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
        "subject": "Sigma Rule 'Suspicious Driver Load from Temp'",
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
(EventID:"6" AND ImageLoaded.keyword:*\\Temp\\*)
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="6" ImageLoaded="*\\Temp\\*")
```


### logpoint
    
```
(event_id="6" ImageLoaded="*\\Temp\\*")
```


### grep
    
```
grep -P '^(?:.*(?=.*6)(?=.*.*\Temp\\.*))'
```



