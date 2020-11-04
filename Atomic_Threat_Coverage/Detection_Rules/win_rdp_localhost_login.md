| Title                    | RDP Login from Localhost       |
|:-------------------------|:------------------|
| **Description**          | RDP login with localhost source address may be a tunnelled login |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1076: Remote Desktop Protocol](https://attack.mitre.org/techniques/T1076)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0004_4624_windows_account_logon](../Data_Needed/DN_0004_4624_windows_account_logon.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1076: Remote Desktop Protocol](../Triggers/T1076.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.fireeye.com/blog/threat-research/2019/01/bypassing-network-restrictions-through-rdp-tunneling.html](https://www.fireeye.com/blog/threat-research/2019/01/bypassing-network-restrictions-through-rdp-tunneling.html)</li></ul>  |
| **Author**               | Thomas Patzke |
| Other Tags           | <ul><li>car.2013-07-002</li></ul> | 

## Detection Rules

### Sigma rule

```
title: RDP Login from Localhost
id: 51e33403-2a37-4d66-a574-1fda1782cc31
description: RDP login with localhost source address may be a tunnelled login
references:
    - https://www.fireeye.com/blog/threat-research/2019/01/bypassing-network-restrictions-through-rdp-tunneling.html
date: 2019/01/28
modified: 2019/01/29
tags:
    - attack.lateral_movement
    - attack.t1076
    - car.2013-07-002
status: experimental
author: Thomas Patzke
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 4624
        LogonType: 10
        SourceNetworkAddress:
            - "::1"
            - "127.0.0.1"
    condition: selection
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent -LogName Security | where {($_.ID -eq "4624" -and $_.message -match "LogonType.*10" -and ($_.message -match "::1" -or $_.message -match "127.0.0.1")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND winlog.event_id:"4624" AND winlog.event_data.LogonType:"10" AND SourceNetworkAddress:("\:\:1" OR "127.0.0.1"))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/51e33403-2a37-4d66-a574-1fda1782cc31 <<EOF
{
  "metadata": {
    "title": "RDP Login from Localhost",
    "description": "RDP login with localhost source address may be a tunnelled login",
    "tags": [
      "attack.lateral_movement",
      "attack.t1076",
      "car.2013-07-002"
    ],
    "query": "(winlog.channel:\"Security\" AND winlog.event_id:\"4624\" AND winlog.event_data.LogonType:\"10\" AND SourceNetworkAddress:(\"\\:\\:1\" OR \"127.0.0.1\"))"
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
                    "query": "(winlog.channel:\"Security\" AND winlog.event_id:\"4624\" AND winlog.event_data.LogonType:\"10\" AND SourceNetworkAddress:(\"\\:\\:1\" OR \"127.0.0.1\"))",
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
        "subject": "Sigma Rule 'RDP Login from Localhost'",
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
(EventID:"4624" AND LogonType:"10" AND SourceNetworkAddress:("\:\:1" "127.0.0.1"))
```


### splunk
    
```
(source="WinEventLog:Security" EventCode="4624" LogonType="10" (SourceNetworkAddress="::1" OR SourceNetworkAddress="127.0.0.1"))
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" event_id="4624" logon_type="10" SourceNetworkAddress IN ["::1", "127.0.0.1"])
```


### grep
    
```
grep -P '^(?:.*(?=.*4624)(?=.*10)(?=.*(?:.*::1|.*127\.0\.0\.1)))'
```



