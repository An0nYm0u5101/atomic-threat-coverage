| Title                    | External Disk Drive or USB Storage Device       |
|:-------------------------|:------------------|
| **Description**          | Detects external diskdrives or plugged in USB devices |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li><li>[TA0001: Initial Access](https://attack.mitre.org/tactics/TA0001)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1091: Replication Through Removable Media](https://attack.mitre.org/techniques/T1091)</li><li>[T1200: Hardware Additions](https://attack.mitre.org/techniques/T1200)</li></ul>  |
| **Data Needed**          |  There is no documented Data Needed for this Detection Rule yet  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | low |
| **False Positives**      | <ul><li>Legitimate administrative activity</li></ul>  |
| **Development Status**   | experimental |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Keith Wright |


## Detection Rules

### Sigma rule

```
title: External Disk Drive or USB Storage Device
id: f69a87ea-955e-4fb4-adb2-bb9fd6685632
description: Detects external diskdrives or plugged in USB devices
status: experimental
author: Keith Wright
date: 2019/11/20
tags:
    - attack.t1091
    - attack.t1200
    - attack.lateral_movement
    - attack.initial_access
logsource:
    product: windows
    service: security
detection:
    selection:
        EventID: 
            - 6416
        DeviceClassName: 'DiskDrive'  
    selection2:
        DeviceDescription: 'USB Mass Storage Device'
    condition: selection or selection2
falsepositives: 
    - Legitimate administrative activity
level: low

```





### powershell
    
```
Get-WinEvent -LogName Security | where {(((($_.ID -eq "6416") -and $_.message -match "DeviceClassName.*DiskDrive") -or $_.message -match "DeviceDescription.*USB Mass Storage Device")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Security" AND ((winlog.event_id:("6416") AND DeviceClassName:"DiskDrive") OR DeviceDescription:"USB\ Mass\ Storage\ Device"))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/f69a87ea-955e-4fb4-adb2-bb9fd6685632 <<EOF
{
  "metadata": {
    "title": "External Disk Drive or USB Storage Device",
    "description": "Detects external diskdrives or plugged in USB devices",
    "tags": [
      "attack.t1091",
      "attack.t1200",
      "attack.lateral_movement",
      "attack.initial_access"
    ],
    "query": "(winlog.channel:\"Security\" AND ((winlog.event_id:(\"6416\") AND DeviceClassName:\"DiskDrive\") OR DeviceDescription:\"USB\\ Mass\\ Storage\\ Device\"))"
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
                    "query": "(winlog.channel:\"Security\" AND ((winlog.event_id:(\"6416\") AND DeviceClassName:\"DiskDrive\") OR DeviceDescription:\"USB\\ Mass\\ Storage\\ Device\"))",
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
        "subject": "Sigma Rule 'External Disk Drive or USB Storage Device'",
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
((EventID:("6416") AND DeviceClassName:"DiskDrive") OR DeviceDescription:"USB Mass Storage Device")
```


### splunk
    
```
(source="WinEventLog:Security" (((EventCode="6416") DeviceClassName="DiskDrive") OR DeviceDescription="USB Mass Storage Device"))
```


### logpoint
    
```
(event_source="Microsoft-Windows-Security-Auditing" ((event_id IN ["6416"] DeviceClassName="DiskDrive") OR DeviceDescription="USB Mass Storage Device"))
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?=.*(?:.*6416))(?=.*DiskDrive))|.*USB Mass Storage Device))'
```



