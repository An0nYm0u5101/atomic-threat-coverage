| Title                    | DNS Exfiltration Tools Execution       |
|:-------------------------|:------------------|
| **Description**          | Well-known DNS Exfiltration tools execution |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0010: Exfiltration](https://attack.mitre.org/tactics/TA0010)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1048: Exfiltration Over Alternative Protocol](https://attack.mitre.org/techniques/T1048)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0001_4688_windows_process_creation](../Data_Needed/DN_0001_4688_windows_process_creation.md)</li><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1048: Exfiltration Over Alternative Protocol](../Triggers/T1048.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Legitimate usage of iodine or dnscat2 — DNS Exfiltration tools (unlikely)</li></ul>  |
| **Development Status**   | experimental |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Daniil Yugoslavskiy, oscd.community |


## Detection Rules

### Sigma rule

```
title: DNS Exfiltration Tools Execution
id: 98a96a5a-64a0-4c42-92c5-489da3866cb0
description: Well-known DNS Exfiltration tools execution
status: experimental
author: Daniil Yugoslavskiy, oscd.community
date: 2019/10/24
tags:
    - attack.exfiltration
    - attack.t1048
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        - Image|endswith: '*\iodine.exe'
        - Image|contains: '\dnscat2'
    condition: selection
falsepositives:
    - Legitimate usage of iodine or dnscat2 — DNS Exfiltration tools (unlikely)
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\iodine.exe" -or $_.message -match "Image.*.*\\dnscat2.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\iodine.exe OR winlog.event_data.Image.keyword:*\\dnscat2*)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/98a96a5a-64a0-4c42-92c5-489da3866cb0 <<EOF
{
  "metadata": {
    "title": "DNS Exfiltration Tools Execution",
    "description": "Well-known DNS Exfiltration tools execution",
    "tags": [
      "attack.exfiltration",
      "attack.t1048"
    ],
    "query": "(winlog.event_data.Image.keyword:*\\\\iodine.exe OR winlog.event_data.Image.keyword:*\\\\dnscat2*)"
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
                    "query": "(winlog.event_data.Image.keyword:*\\\\iodine.exe OR winlog.event_data.Image.keyword:*\\\\dnscat2*)",
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
        "subject": "Sigma Rule 'DNS Exfiltration Tools Execution'",
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
(Image.keyword:*\\iodine.exe OR Image.keyword:*\\dnscat2*)
```


### splunk
    
```
(Image="*\\iodine.exe" OR Image="*\\dnscat2*")
```


### logpoint
    
```
(Image="*\\iodine.exe" OR Image="*\\dnscat2*")
```


### grep
    
```
grep -P '^(?:.*(?:.*.*\iodine\.exe|.*.*\dnscat2.*))'
```



