| Title                    | Suspicious Service Path Modification       |
|:-------------------------|:------------------|
| **Description**          | Detects service path modification to powershell/cmd |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li><li>[TA0004: Privilege Escalation](https://attack.mitre.org/tactics/TA0004)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1543.003: Windows Service](https://attack.mitre.org/techniques/T1543/003)</li><li>[T1031: Modify Existing Service](https://attack.mitre.org/techniques/T1031)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1543.003: Windows Service](../Triggers/T1543.003.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1031/T1031.yaml](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1031/T1031.yaml)</li></ul>  |
| **Author**               | Victor Sergeev, oscd.community |


## Detection Rules

### Sigma rule

```
title: Suspicious Service Path Modification
id: 138d3531-8793-4f50-a2cd-f291b2863d78
description: Detects service path modification to powershell/cmd
status: experimental
references:
    - https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1031/T1031.yaml
tags:
    - attack.persistence
    - attack.privilege_escalation
    - attack.t1543.003
    - attack.t1031      # an old one     
date: 2019/10/21
modified: 2020/08/28
author: Victor Sergeev, oscd.community
logsource:
    category: process_creation
    product: windows
detection:
    selection_1:
        Image|endswith: '\sc.exe'
        CommandLine|contains|all:
            - 'config'
            - 'binpath'
    selection_2:
        CommandLine|contains:
            - 'powershell'
            - 'cmd'
    condition: selection_1 and selection_2
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Unknown
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\\\sc.exe" -and $_.message -match "CommandLine.*.*config.*" -and $_.message -match "CommandLine.*.*binpath.*" -and ($_.message -match "CommandLine.*.*powershell.*" -or $_.message -match "CommandLine.*.*cmd.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\\\sc.exe AND winlog.event_data.CommandLine.keyword:*config* AND winlog.event_data.CommandLine.keyword:*binpath* AND winlog.event_data.CommandLine.keyword:(*powershell* OR *cmd*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/138d3531-8793-4f50-a2cd-f291b2863d78 <<EOF\n{\n  "metadata": {\n    "title": "Suspicious Service Path Modification",\n    "description": "Detects service path modification to powershell/cmd",\n    "tags": [\n      "attack.persistence",\n      "attack.privilege_escalation",\n      "attack.t1543.003",\n      "attack.t1031"\n    ],\n    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\sc.exe AND winlog.event_data.CommandLine.keyword:*config* AND winlog.event_data.CommandLine.keyword:*binpath* AND winlog.event_data.CommandLine.keyword:(*powershell* OR *cmd*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\sc.exe AND winlog.event_data.CommandLine.keyword:*config* AND winlog.event_data.CommandLine.keyword:*binpath* AND winlog.event_data.CommandLine.keyword:(*powershell* OR *cmd*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Suspicious Service Path Modification\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:*\\\\sc.exe AND CommandLine.keyword:*config* AND CommandLine.keyword:*binpath* AND CommandLine.keyword:(*powershell* *cmd*))
```


### splunk
    
```
(Image="*\\\\sc.exe" CommandLine="*config*" CommandLine="*binpath*" (CommandLine="*powershell*" OR CommandLine="*cmd*")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
(Image="*\\\\sc.exe" CommandLine="*config*" CommandLine="*binpath*" CommandLine IN ["*powershell*", "*cmd*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\sc\\.exe)(?=.*.*config.*)(?=.*.*binpath.*)(?=.*(?:.*.*powershell.*|.*.*cmd.*)))'
```



