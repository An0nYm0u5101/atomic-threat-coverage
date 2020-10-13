| Title                    | WCE wceaux.dll Access       |
|:-------------------------|:------------------|
| **Description**          | Detects wceaux.dll access while WCE pass-the-hash remote command execution on source host |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0058_4656_handle_to_an_object_was_requested](../Data_Needed/DN_0058_4656_handle_to_an_object_was_requested.md)</li><li>[DN_0060_4658_handle_to_an_object_was_closed](../Data_Needed/DN_0060_4658_handle_to_an_object_was_closed.md)</li><li>[DN_0061_4660_object_was_deleted](../Data_Needed/DN_0061_4660_object_was_deleted.md)</li><li>[DN_0062_4663_attempt_was_made_to_access_an_object](../Data_Needed/DN_0062_4663_attempt_was_made_to_access_an_object.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Penetration testing</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.jpcert.or.jp/english/pub/sr/ir_research.html](https://www.jpcert.or.jp/english/pub/sr/ir_research.html)</li><li>[https://jpcertcc.github.io/ToolAnalysisResultSheet](https://jpcertcc.github.io/ToolAnalysisResultSheet)</li></ul>  |
| **Author**               | Thomas Patzke |
| Other Tags           | <ul><li>attack.s0005</li></ul> | 

## Detection Rules

### Sigma rule

```
title: WCE wceaux.dll Access
id: 1de68c67-af5c-4097-9c85-fe5578e09e67
status: experimental
description: Detects wceaux.dll access while WCE pass-the-hash remote command execution on source host
author: Thomas Patzke
date: 2017/06/14
references:
    - https://www.jpcert.or.jp/english/pub/sr/ir_research.html
    - https://jpcertcc.github.io/ToolAnalysisResultSheet
tags:
    - attack.credential_access
    - attack.t1003
    - attack.s0005
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID:
            - 4656
            - 4658
            - 4660
            - 4663
        ObjectName: '*\wceaux.dll'
    condition: selection
falsepositives:
    - Penetration testing
level: critical

```





### powershell
    
```
Get-WinEvent -LogName Security | where {(($_.ID -eq "4656" -or $_.ID -eq "4658" -or $_.ID -eq "4660" -or $_.ID -eq "4663") -and $_.message -match "ObjectName.*.*\\wceaux.dll") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND winlog.event_id:("4656" OR "4658" OR "4660" OR "4663") AND winlog.event_data.ObjectName.keyword:*\\wceaux.dll)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/1de68c67-af5c-4097-9c85-fe5578e09e67 <<EOF
{
  "metadata": {
    "title": "WCE wceaux.dll Access",
    "description": "Detects wceaux.dll access while WCE pass-the-hash remote command execution on source host",
    "tags": [
      "attack.credential_access",
      "attack.t1003",
      "attack.s0005"
    ],
    "query": "(winlog.channel:\"Security\" AND winlog.event_id:(\"4656\" OR \"4658\" OR \"4660\" OR \"4663\") AND winlog.event_data.ObjectName.keyword:*\\\\wceaux.dll)"
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
                    "query": "(winlog.channel:\"Security\" AND winlog.event_id:(\"4656\" OR \"4658\" OR \"4660\" OR \"4663\") AND winlog.event_data.ObjectName.keyword:*\\\\wceaux.dll)",
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
        "subject": "Sigma Rule 'WCE wceaux.dll Access'",
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
(EventID:("4656" "4658" "4660" "4663") AND ObjectName.keyword:*\\wceaux.dll)
```


### splunk
    
```
(source="WinEventLog:Security" (EventCode="4656" OR EventCode="4658" OR EventCode="4660" OR EventCode="4663") ObjectName="*\\wceaux.dll")
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" event_id IN ["4656", "4658", "4660", "4663"] ObjectName="*\\wceaux.dll")
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*4656|.*4658|.*4660|.*4663))(?=.*.*\wceaux\.dll))'
```



