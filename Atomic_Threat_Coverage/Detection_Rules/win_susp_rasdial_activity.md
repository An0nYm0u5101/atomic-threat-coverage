| Title                    | Suspicious RASdial Activity       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious process related to rasdial.exe |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059: Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059)</li><li>[T1064: Scripting](https://attack.mitre.org/techniques/T1064)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0001_4688_windows_process_creation](../Data_Needed/DN_0001_4688_windows_process_creation.md)</li><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>False positives depend on scripts and administrative tools used in the monitored environment</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://twitter.com/subTee/status/891298217907830785](https://twitter.com/subTee/status/891298217907830785)</li></ul>  |
| **Author**               | juju4 |


## Detection Rules

### Sigma rule

```
title: Suspicious RASdial Activity
id: 6bba49bf-7f8c-47d6-a1bb-6b4dece4640e
description: Detects suspicious process related to rasdial.exe
status: experimental
references:
    - https://twitter.com/subTee/status/891298217907830785
author: juju4
date: 2019/01/16
tags:
    - attack.defense_evasion
    - attack.execution
    - attack.t1059
    - attack.t1064      # an old one 
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith:
            - rasdial.exe
    condition: selection
falsepositives:
    - False positives depend on scripts and administrative tools used in the monitored environment
level: medium

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*rasdial.exe") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.Image.keyword:(*rasdial.exe)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/6bba49bf-7f8c-47d6-a1bb-6b4dece4640e <<EOF
{
  "metadata": {
    "title": "Suspicious RASdial Activity",
    "description": "Detects suspicious process related to rasdial.exe",
    "tags": [
      "attack.defense_evasion",
      "attack.execution",
      "attack.t1059",
      "attack.t1064"
    ],
    "query": "winlog.event_data.Image.keyword:(*rasdial.exe)"
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
                    "query": "winlog.event_data.Image.keyword:(*rasdial.exe)",
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
        "subject": "Sigma Rule 'Suspicious RASdial Activity'",
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
Image.keyword:(*rasdial.exe)
```


### splunk
    
```
(Image="*rasdial.exe")
```


### logpoint
    
```
Image IN ["*rasdial.exe"]
```


### grep
    
```
grep -P '^(?:.*.*rasdial\.exe)'
```



