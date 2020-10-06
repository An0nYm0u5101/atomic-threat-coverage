| Title                    | Fireball Archer Install       |
|:-------------------------|:------------------|
| **Description**          | Detects Archer malware invocation via rundll32 |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1218.011: Rundll32](https://attack.mitre.org/techniques/T1218/011)</li><li>[T1085: Rundll32](https://attack.mitre.org/techniques/T1085)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1218.011: Rundll32](../Triggers/T1218.011.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.virustotal.com/en/file/9b4971349ae85aa09c0a69852ed3e626c954954a3927b3d1b6646f139b930022/analysis/](https://www.virustotal.com/en/file/9b4971349ae85aa09c0a69852ed3e626c954954a3927b3d1b6646f139b930022/analysis/)</li><li>[https://www.hybrid-analysis.com/sample/9b4971349ae85aa09c0a69852ed3e626c954954a3927b3d1b6646f139b930022?environmentId=100](https://www.hybrid-analysis.com/sample/9b4971349ae85aa09c0a69852ed3e626c954954a3927b3d1b6646f139b930022?environmentId=100)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Fireball Archer Install
id: 3d4aebe0-6d29-45b2-a8a4-3dfde586a26d
status: experimental
description: Detects Archer malware invocation via rundll32
author: Florian Roth
date: 2017/06/03
modified: 2020/08/29
references:
    - https://www.virustotal.com/en/file/9b4971349ae85aa09c0a69852ed3e626c954954a3927b3d1b6646f139b930022/analysis/
    - https://www.hybrid-analysis.com/sample/9b4971349ae85aa09c0a69852ed3e626c954954a3927b3d1b6646f139b930022?environmentId=100
tags:
    - attack.execution
    - attack.defense_evasion
    - attack.t1218.011
    - attack.t1085  # an old one
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine: '*\rundll32.exe *,InstallArcherSvc'
    condition: selection
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {$_.message -match "CommandLine.*.*\\\\rundll32.exe .*,InstallArcherSvc" } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:*\\\\rundll32.exe\\ *,InstallArcherSvc
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/3d4aebe0-6d29-45b2-a8a4-3dfde586a26d <<EOF\n{\n  "metadata": {\n    "title": "Fireball Archer Install",\n    "description": "Detects Archer malware invocation via rundll32",\n    "tags": [\n      "attack.execution",\n      "attack.defense_evasion",\n      "attack.t1218.011",\n      "attack.t1085"\n    ],\n    "query": "winlog.event_data.CommandLine.keyword:*\\\\\\\\rundll32.exe\\\\ *,InstallArcherSvc"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "winlog.event_data.CommandLine.keyword:*\\\\\\\\rundll32.exe\\\\ *,InstallArcherSvc",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Fireball Archer Install\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
CommandLine.keyword:*\\\\rundll32.exe *,InstallArcherSvc
```


### splunk
    
```
CommandLine="*\\\\rundll32.exe *,InstallArcherSvc" | table CommandLine,ParentCommandLine
```


### logpoint
    
```
CommandLine="*\\\\rundll32.exe *,InstallArcherSvc"
```


### grep
    
```
grep -P '^.*\\rundll32\\.exe .*,InstallArcherSvc'
```



