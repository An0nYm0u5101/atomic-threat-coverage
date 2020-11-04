| Title                    | Turla PNG Dropper Service       |
|:-------------------------|:------------------|
| **Description**          | This method detects malicious services mentioned in Turla PNG dropper report by NCC Group in November 2018 |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1050: New Service](https://attack.mitre.org/techniques/T1050)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0005_7045_windows_service_insatalled](../Data_Needed/DN_0005_7045_windows_service_insatalled.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1050: New Service](../Triggers/T1050.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>unlikely</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           | <ul><li>[https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2018/november/turla-png-dropper-is-back/](https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2018/november/turla-png-dropper-is-back/)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>attack.g0010</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Turla PNG Dropper Service
id: 1228f8e2-7e79-4dea-b0ad-c91f1d5016c1
description: This method detects malicious services mentioned in Turla PNG dropper report by NCC Group in November 2018
references:
    - https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2018/november/turla-png-dropper-is-back/
author: Florian Roth
date: 2018/11/23
tags:
    - attack.persistence
    - attack.g0010
    - attack.t1050
logsource:
    product: windows
    service: system
detection:
    selection:
        EventID: 7045
        ServiceName: 'WerFaultSvc'
    condition: selection
falsepositives:
    - unlikely
level: critical

```





### powershell
    
```
Get-WinEvent -LogName System | where {($_.ID -eq "7045" -and $_.message -match "ServiceName.*WerFaultSvc") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_id:"7045" AND winlog.event_data.ServiceName:"WerFaultSvc")
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/1228f8e2-7e79-4dea-b0ad-c91f1d5016c1 <<EOF
{
  "metadata": {
    "title": "Turla PNG Dropper Service",
    "description": "This method detects malicious services mentioned in Turla PNG dropper report by NCC Group in November 2018",
    "tags": [
      "attack.persistence",
      "attack.g0010",
      "attack.t1050"
    ],
    "query": "(winlog.event_id:\"7045\" AND winlog.event_data.ServiceName:\"WerFaultSvc\")"
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
                    "query": "(winlog.event_id:\"7045\" AND winlog.event_data.ServiceName:\"WerFaultSvc\")",
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
        "subject": "Sigma Rule 'Turla PNG Dropper Service'",
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
(EventID:"7045" AND ServiceName:"WerFaultSvc")
```


### splunk
    
```
(source="WinEventLog:System" EventCode="7045" ServiceName="WerFaultSvc")
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" event_id="7045" service="WerFaultSvc")
```


### grep
    
```
grep -P '^(?:.*(?=.*7045)(?=.*WerFaultSvc))'
```



