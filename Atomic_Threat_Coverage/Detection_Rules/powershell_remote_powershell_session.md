| Title                    | Remote PowerShell Session       |
|:-------------------------|:------------------|
| **Description**          | Detects remote PowerShell sessions |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li><li>[TA0008: Lateral Movement](https://attack.mitre.org/tactics/TA0008)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li><li>[T1021.006: Windows Remote Management](https://attack.mitre.org/techniques/T1021/006)</li><li>[T1028: Windows Remote Management](https://attack.mitre.org/techniques/T1028)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0037_4103_windows_powershell_executing_pipeline](../Data_Needed/DN_0037_4103_windows_powershell_executing_pipeline.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li><li>[T1021.006: Windows Remote Management](../Triggers/T1021.006.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Legitimate use remote PowerShell sessions</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1086_powershell/powershell_remote_session.md](https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1086_powershell/powershell_remote_session.md)</li></ul>  |
| **Author**               | Roberto Rodriguez @Cyb3rWard0g |


## Detection Rules

### Sigma rule

```
title: Remote PowerShell Session
id: 96b9f619-aa91-478f-bacb-c3e50f8df575
description: Detects remote PowerShell sessions
status: experimental
date: 2019/08/10
modified: 2020/08/24
author: Roberto Rodriguez @Cyb3rWard0g
references:
    - https://github.com/Cyb3rWard0g/ThreatHunter-Playbook/tree/master/playbooks/windows/02_execution/T1086_powershell/powershell_remote_session.md
tags:
    - attack.execution
    - attack.t1059.001
    - attack.t1086  #an old one
    - attack.lateral_movement
    - attack.t1021.006
    - attack.t1028  #an old one
logsource:
    product: windows
    service: powershell
detection:
    selection:
        EventID:
            - 4103
            - 400
        HostName: 'ServerRemoteHost'
        HostApplication|contains: 'wsmprovhost.exe'
    condition: selection
falsepositives:
    - Legitimate use remote PowerShell sessions
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-PowerShell/Operational | where {(($_.ID -eq "4103" -or $_.ID -eq "400") -and $_.message -match "HostName.*ServerRemoteHost" -and $_.message -match "HostApplication.*.*wsmprovhost.exe.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_id:("4103" OR "400") AND HostName:"ServerRemoteHost" AND HostApplication.keyword:*wsmprovhost.exe*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/96b9f619-aa91-478f-bacb-c3e50f8df575 <<EOF\n{\n  "metadata": {\n    "title": "Remote PowerShell Session",\n    "description": "Detects remote PowerShell sessions",\n    "tags": [\n      "attack.execution",\n      "attack.t1059.001",\n      "attack.t1086",\n      "attack.lateral_movement",\n      "attack.t1021.006",\n      "attack.t1028"\n    ],\n    "query": "(winlog.event_id:(\\"4103\\" OR \\"400\\") AND HostName:\\"ServerRemoteHost\\" AND HostApplication.keyword:*wsmprovhost.exe*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_id:(\\"4103\\" OR \\"400\\") AND HostName:\\"ServerRemoteHost\\" AND HostApplication.keyword:*wsmprovhost.exe*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Remote PowerShell Session\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:("4103" "400") AND HostName:"ServerRemoteHost" AND HostApplication.keyword:*wsmprovhost.exe*)
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-PowerShell/Operational" (EventCode="4103" OR EventCode="400") HostName="ServerRemoteHost" HostApplication="*wsmprovhost.exe*")
```


### logpoint
    
```
(event_id IN ["4103", "400"] HostName="ServerRemoteHost" HostApplication="*wsmprovhost.exe*")
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*4103|.*400))(?=.*ServerRemoteHost)(?=.*.*wsmprovhost\\.exe.*))'
```



