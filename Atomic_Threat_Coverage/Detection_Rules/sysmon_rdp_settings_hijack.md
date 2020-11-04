| Title                    | RDP Sensitive Settings Changed       |
|:-------------------------|:------------------|
| **Description**          | Detects changes to RDP terminal service sensitive settings |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** |  This Detection Rule wasn't mapped to ATT&amp;CK Technique yet  |
| **Data Needed**          | <ul><li>[DN_0017_13_windows_sysmon_RegistryEvent](../Data_Needed/DN_0017_13_windows_sysmon_RegistryEvent.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           | <ul><li>[https://blog.menasec.net/2019/02/threat-hunting-rdp-hijacking-via.html](https://blog.menasec.net/2019/02/threat-hunting-rdp-hijacking-via.html)</li></ul>  |
| **Author**               | Samir Bousseaden |


## Detection Rules

### Sigma rule

```
title: RDP Sensitive Settings Changed
id: 171b67e1-74b4-460e-8d55-b331f3e32d67
description: Detects changes to RDP terminal service sensitive settings
references:
    - https://blog.menasec.net/2019/02/threat-hunting-rdp-hijacking-via.html
date: 2019/04/03
author: Samir Bousseaden
logsource:
    product: windows
    service: sysmon
detection:
    selection_reg:
        EventID: 13
        TargetObject:
            - '*\services\TermService\Parameters\ServiceDll*'
            - '*\Control\Terminal Server\fSingleSessionPerUser*'
            - '*\Control\Terminal Server\fDenyTSConnections*'
    condition: selection_reg
tags:
    - attack.defense_evasion
falsepositives:
    - unknown
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "13" -and ($_.message -match "TargetObject.*.*\\services\\TermService\\Parameters\\ServiceDll.*" -or $_.message -match "TargetObject.*.*\\Control\\Terminal Server\\fSingleSessionPerUser.*" -or $_.message -match "TargetObject.*.*\\Control\\Terminal Server\\fDenyTSConnections.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND winlog.event_id:"13" AND winlog.event_data.TargetObject.keyword:(*\\services\\TermService\\Parameters\\ServiceDll* OR *\\Control\\Terminal\ Server\\fSingleSessionPerUser* OR *\\Control\\Terminal\ Server\\fDenyTSConnections*))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/171b67e1-74b4-460e-8d55-b331f3e32d67 <<EOF
{
  "metadata": {
    "title": "RDP Sensitive Settings Changed",
    "description": "Detects changes to RDP terminal service sensitive settings",
    "tags": [
      "attack.defense_evasion"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"13\" AND winlog.event_data.TargetObject.keyword:(*\\\\services\\\\TermService\\\\Parameters\\\\ServiceDll* OR *\\\\Control\\\\Terminal\\ Server\\\\fSingleSessionPerUser* OR *\\\\Control\\\\Terminal\\ Server\\\\fDenyTSConnections*))"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"13\" AND winlog.event_data.TargetObject.keyword:(*\\\\services\\\\TermService\\\\Parameters\\\\ServiceDll* OR *\\\\Control\\\\Terminal\\ Server\\\\fSingleSessionPerUser* OR *\\\\Control\\\\Terminal\\ Server\\\\fDenyTSConnections*))",
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
        "subject": "Sigma Rule 'RDP Sensitive Settings Changed'",
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
(EventID:"13" AND TargetObject.keyword:(*\\services\\TermService\\Parameters\\ServiceDll* *\\Control\\Terminal Server\\fSingleSessionPerUser* *\\Control\\Terminal Server\\fDenyTSConnections*))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="13" (TargetObject="*\\services\\TermService\\Parameters\\ServiceDll*" OR TargetObject="*\\Control\\Terminal Server\\fSingleSessionPerUser*" OR TargetObject="*\\Control\\Terminal Server\\fDenyTSConnections*"))
```


### logpoint
    
```
(event_id="13" TargetObject IN ["*\\services\\TermService\\Parameters\\ServiceDll*", "*\\Control\\Terminal Server\\fSingleSessionPerUser*", "*\\Control\\Terminal Server\\fDenyTSConnections*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*13)(?=.*(?:.*.*\services\TermService\Parameters\ServiceDll.*|.*.*\Control\Terminal Server\fSingleSessionPerUser.*|.*.*\Control\Terminal Server\fDenyTSConnections.*)))'
```



