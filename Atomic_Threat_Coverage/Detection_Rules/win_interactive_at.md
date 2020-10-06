| Title                    | Interactive AT Job       |
|:-------------------------|:------------------|
| **Description**          | Detect an interactive AT job, which may be used as a form of privilege escalation |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0004: Privilege Escalation](https://attack.mitre.org/tactics/TA0004)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1053.002: At (Windows)](https://attack.mitre.org/techniques/T1053/002)</li><li>[T1053: Scheduled Task/Job](https://attack.mitre.org/techniques/T1053)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1053.002: At (Windows)](../Triggers/T1053.002.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unlikely (at.exe deprecated as of Windows 8)</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1053/T1053.yaml](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1053/T1053.yaml)</li><li>[https://eqllib.readthedocs.io/en/latest/analytics/d8db43cf-ed52-4f5c-9fb3-c9a4b95a0b56.html](https://eqllib.readthedocs.io/en/latest/analytics/d8db43cf-ed52-4f5c-9fb3-c9a4b95a0b56.html)</li></ul>  |
| **Author**               | E.M. Anhaus (orignally from Atomic Blue Detections, Endgame), oscd.community |


## Detection Rules

### Sigma rule

```
title: Interactive AT Job
id: 60fc936d-2eb0-4543-8a13-911c750a1dfc
description: Detect an interactive AT job, which may be used as a form of privilege escalation
status: experimental
author: E.M. Anhaus (orignally from Atomic Blue Detections, Endgame), oscd.community
references:
    - https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1053/T1053.yaml
    - https://eqllib.readthedocs.io/en/latest/analytics/d8db43cf-ed52-4f5c-9fb3-c9a4b95a0b56.html
date: 2019/10/24
modified: 2019/11/11
tags:
    - attack.privilege_escalation
    - attack.t1053.002
    - attack.t1053  # an old one
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\at.exe'
        CommandLine|contains: 'interactive'
    condition: selection
fields:
    - ComputerName
    - User
    - CommandLine
falsepositives:
    - Unlikely (at.exe deprecated as of Windows 8)
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\\\at.exe" -and $_.message -match "CommandLine.*.*interactive.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\\\at.exe AND winlog.event_data.CommandLine.keyword:*interactive*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/60fc936d-2eb0-4543-8a13-911c750a1dfc <<EOF\n{\n  "metadata": {\n    "title": "Interactive AT Job",\n    "description": "Detect an interactive AT job, which may be used as a form of privilege escalation",\n    "tags": [\n      "attack.privilege_escalation",\n      "attack.t1053.002",\n      "attack.t1053"\n    ],\n    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\at.exe AND winlog.event_data.CommandLine.keyword:*interactive*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\at.exe AND winlog.event_data.CommandLine.keyword:*interactive*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Interactive AT Job\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\nComputerName = {{_source.ComputerName}}\\n        User = {{_source.User}}\\n CommandLine = {{_source.CommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:*\\\\at.exe AND CommandLine.keyword:*interactive*)
```


### splunk
    
```
(Image="*\\\\at.exe" CommandLine="*interactive*") | table ComputerName,User,CommandLine
```


### logpoint
    
```
(Image="*\\\\at.exe" CommandLine="*interactive*")
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\at\\.exe)(?=.*.*interactive.*))'
```



