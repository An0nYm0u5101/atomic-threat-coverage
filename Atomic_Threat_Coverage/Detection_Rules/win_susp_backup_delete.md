| Title                    | Backup Catalog Deleted       |
|:-------------------------|:------------------|
| **Description**          | Detects backup catalog deletions |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1107: File Deletion](https://attack.mitre.org/techniques/T1107)</li><li>[T1070.004: File Deletion](https://attack.mitre.org/techniques/T1070/004)</li></ul>  |
| **Data Needed**          |  There is no documented Data Needed for this Detection Rule yet  |
| **Trigger**              | <ul><li>[T1070.004: File Deletion](../Triggers/T1070.004.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://technet.microsoft.com/en-us/library/cc742154(v=ws.11).aspx](https://technet.microsoft.com/en-us/library/cc742154(v=ws.11).aspx)</li><li>[https://www.hybrid-analysis.com/sample/ed01ebfbc9eb5bbea545af4d01bf5f1071661840480439c6e5babe8e080e41aa?environmentId=100](https://www.hybrid-analysis.com/sample/ed01ebfbc9eb5bbea545af4d01bf5f1071661840480439c6e5babe8e080e41aa?environmentId=100)</li></ul>  |
| **Author**               | Florian Roth (rule), Tom U. @c_APT_ure (collection) |


## Detection Rules

### Sigma rule

```
title: Backup Catalog Deleted
id: 9703792d-fd9a-456d-a672-ff92efe4806a
status: experimental
description: Detects backup catalog deletions
references:
    - https://technet.microsoft.com/en-us/library/cc742154(v=ws.11).aspx
    - https://www.hybrid-analysis.com/sample/ed01ebfbc9eb5bbea545af4d01bf5f1071661840480439c6e5babe8e080e41aa?environmentId=100
author: Florian Roth (rule), Tom U. @c_APT_ure (collection)
date: 2017/05/12
tags:
    - attack.defense_evasion
    - attack.t1107          # an old one
    - attack.t1070.004
logsource:
    product: windows
    service: application
detection:
    selection:
        EventID: 524
        Source: Microsoft-Windows-Backup
    condition: selection
falsepositives:
    - Unknown
level: medium

```





### powershell
    
```
Get-WinEvent -LogName Application | where {($_.ID -eq "524" -and $_.message -match "Source.*Microsoft-Windows-Backup") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Application" AND winlog.event_id:"524" AND winlog.event_data.Source:"Microsoft\\-Windows\\-Backup")
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/9703792d-fd9a-456d-a672-ff92efe4806a <<EOF\n{\n  "metadata": {\n    "title": "Backup Catalog Deleted",\n    "description": "Detects backup catalog deletions",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1107",\n      "attack.t1070.004"\n    ],\n    "query": "(winlog.channel:\\"Application\\" AND winlog.event_id:\\"524\\" AND winlog.event_data.Source:\\"Microsoft\\\\-Windows\\\\-Backup\\")"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Application\\" AND winlog.event_id:\\"524\\" AND winlog.event_data.Source:\\"Microsoft\\\\-Windows\\\\-Backup\\")",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Backup Catalog Deleted\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"524" AND Source:"Microsoft\\-Windows\\-Backup")
```


### splunk
    
```
(source="WinEventLog:Application" EventCode="524" Source="Microsoft-Windows-Backup")
```


### logpoint
    
```
(event_id="524" Source="Microsoft-Windows-Backup")
```


### grep
    
```
grep -P '^(?:.*(?=.*524)(?=.*Microsoft-Windows-Backup))'
```



