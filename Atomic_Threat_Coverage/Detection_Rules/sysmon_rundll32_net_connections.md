| Title                    | Rundll32 Internet Connection       |
|:-------------------------|:------------------|
| **Description**          | Detects a rundll32 that communicates with public IP addresses |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1085: Rundll32](https://attack.mitre.org/techniques/T1085)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0007_3_windows_sysmon_network_connection](../Data_Needed/DN_0007_3_windows_sysmon_network_connection.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1085: Rundll32](../Triggers/T1085.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Communication to other corporate systems that use IP addresses from public address spaces</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.hybrid-analysis.com/sample/759fb4c0091a78c5ee035715afe3084686a8493f39014aea72dae36869de9ff6?environmentId=100](https://www.hybrid-analysis.com/sample/759fb4c0091a78c5ee035715afe3084686a8493f39014aea72dae36869de9ff6?environmentId=100)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Rundll32 Internet Connection
id: cdc8da7d-c303-42f8-b08c-b4ab47230263
status: experimental
description: Detects a rundll32 that communicates with public IP addresses
references:
    - https://www.hybrid-analysis.com/sample/759fb4c0091a78c5ee035715afe3084686a8493f39014aea72dae36869de9ff6?environmentId=100
author: Florian Roth
date: 2017/11/04
tags:
    - attack.t1085
    - attack.defense_evasion
    - attack.execution
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 3
        Image: '*\rundll32.exe'
        Initiated: 'true'
    filter:
        DestinationIp: 
            - '10.*'
            - '192.168.*'
            - '172.16.*'
            - '172.17.*'
            - '172.18.*'
            - '172.19.*'
            - '172.20.*'
            - '172.21.*'
            - '172.22.*'
            - '172.23.*'
            - '172.24.*'
            - '172.25.*'
            - '172.26.*'
            - '172.27.*'
            - '172.28.*'
            - '172.29.*'
            - '172.30.*'
            - '172.31.*'
            - '127.*'
    condition: selection and not filter
falsepositives:
    - Communication to other corporate systems that use IP addresses from public address spaces
level: medium

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {(($_.ID -eq "3" -and $_.message -match "Image.*.*\\rundll32.exe" -and $_.message -match "Initiated.*true") -and  -not (($_.message -match "DestinationIp.*10..*" -or $_.message -match "DestinationIp.*192.168..*" -or $_.message -match "DestinationIp.*172.16..*" -or $_.message -match "DestinationIp.*172.17..*" -or $_.message -match "DestinationIp.*172.18..*" -or $_.message -match "DestinationIp.*172.19..*" -or $_.message -match "DestinationIp.*172.20..*" -or $_.message -match "DestinationIp.*172.21..*" -or $_.message -match "DestinationIp.*172.22..*" -or $_.message -match "DestinationIp.*172.23..*" -or $_.message -match "DestinationIp.*172.24..*" -or $_.message -match "DestinationIp.*172.25..*" -or $_.message -match "DestinationIp.*172.26..*" -or $_.message -match "DestinationIp.*172.27..*" -or $_.message -match "DestinationIp.*172.28..*" -or $_.message -match "DestinationIp.*172.29..*" -or $_.message -match "DestinationIp.*172.30..*" -or $_.message -match "DestinationIp.*172.31..*" -or $_.message -match "DestinationIp.*127..*"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND (winlog.event_id:"3" AND winlog.event_data.Image.keyword:*\\rundll32.exe AND Initiated:"true") AND (NOT (winlog.event_data.DestinationIp.keyword:(10.* OR 192.168.* OR 172.16.* OR 172.17.* OR 172.18.* OR 172.19.* OR 172.20.* OR 172.21.* OR 172.22.* OR 172.23.* OR 172.24.* OR 172.25.* OR 172.26.* OR 172.27.* OR 172.28.* OR 172.29.* OR 172.30.* OR 172.31.* OR 127.*))))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/cdc8da7d-c303-42f8-b08c-b4ab47230263 <<EOF
{
  "metadata": {
    "title": "Rundll32 Internet Connection",
    "description": "Detects a rundll32 that communicates with public IP addresses",
    "tags": [
      "attack.t1085",
      "attack.defense_evasion",
      "attack.execution"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"3\" AND winlog.event_data.Image.keyword:*\\\\rundll32.exe AND Initiated:\"true\") AND (NOT (winlog.event_data.DestinationIp.keyword:(10.* OR 192.168.* OR 172.16.* OR 172.17.* OR 172.18.* OR 172.19.* OR 172.20.* OR 172.21.* OR 172.22.* OR 172.23.* OR 172.24.* OR 172.25.* OR 172.26.* OR 172.27.* OR 172.28.* OR 172.29.* OR 172.30.* OR 172.31.* OR 127.*))))"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND (winlog.event_id:\"3\" AND winlog.event_data.Image.keyword:*\\\\rundll32.exe AND Initiated:\"true\") AND (NOT (winlog.event_data.DestinationIp.keyword:(10.* OR 192.168.* OR 172.16.* OR 172.17.* OR 172.18.* OR 172.19.* OR 172.20.* OR 172.21.* OR 172.22.* OR 172.23.* OR 172.24.* OR 172.25.* OR 172.26.* OR 172.27.* OR 172.28.* OR 172.29.* OR 172.30.* OR 172.31.* OR 127.*))))",
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
        "subject": "Sigma Rule 'Rundll32 Internet Connection'",
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
((EventID:"3" AND Image.keyword:*\\rundll32.exe AND Initiated:"true") AND (NOT (DestinationIp.keyword:(10.* 192.168.* 172.16.* 172.17.* 172.18.* 172.19.* 172.20.* 172.21.* 172.22.* 172.23.* 172.24.* 172.25.* 172.26.* 172.27.* 172.28.* 172.29.* 172.30.* 172.31.* 127.*))))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode="3" Image="*\\rundll32.exe" Initiated="true") NOT ((DestinationIp="10.*" OR DestinationIp="192.168.*" OR DestinationIp="172.16.*" OR DestinationIp="172.17.*" OR DestinationIp="172.18.*" OR DestinationIp="172.19.*" OR DestinationIp="172.20.*" OR DestinationIp="172.21.*" OR DestinationIp="172.22.*" OR DestinationIp="172.23.*" OR DestinationIp="172.24.*" OR DestinationIp="172.25.*" OR DestinationIp="172.26.*" OR DestinationIp="172.27.*" OR DestinationIp="172.28.*" OR DestinationIp="172.29.*" OR DestinationIp="172.30.*" OR DestinationIp="172.31.*" OR DestinationIp="127.*")))
```


### logpoint
    
```
((event_id="3" Image="*\\rundll32.exe" Initiated="true")  -(DestinationIp IN ["10.*", "192.168.*", "172.16.*", "172.17.*", "172.18.*", "172.19.*", "172.20.*", "172.21.*", "172.22.*", "172.23.*", "172.24.*", "172.25.*", "172.26.*", "172.27.*", "172.28.*", "172.29.*", "172.30.*", "172.31.*", "127.*"]))
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*(?=.*3)(?=.*.*\rundll32\.exe)(?=.*true)))(?=.*(?!.*(?:.*(?=.*(?:.*10\..*|.*192\.168\..*|.*172\.16\..*|.*172\.17\..*|.*172\.18\..*|.*172\.19\..*|.*172\.20\..*|.*172\.21\..*|.*172\.22\..*|.*172\.23\..*|.*172\.24\..*|.*172\.25\..*|.*172\.26\..*|.*172\.27\..*|.*172\.28\..*|.*172\.29\..*|.*172\.30\..*|.*172\.31\..*|.*127\..*))))))'
```



