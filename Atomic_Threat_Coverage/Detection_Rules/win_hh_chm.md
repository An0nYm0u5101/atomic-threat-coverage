| Title                    | HH.exe Execution       |
|:-------------------------|:------------------|
| **Description**          | Identifies usage of hh.exe executing recently modified .chm files. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1218.001: Compiled HTML File](https://attack.mitre.org/techniques/T1218/001)</li><li>[T1223: Compiled HTML File](https://attack.mitre.org/techniques/T1223)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1218.001: Compiled HTML File](../Triggers/T1218.001.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unlike</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1223/T1223.yaml](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1223/T1223.yaml)</li><li>[https://eqllib.readthedocs.io/en/latest/analytics/b25aa548-7937-11e9-8f5c-d46d6d62a49e.html](https://eqllib.readthedocs.io/en/latest/analytics/b25aa548-7937-11e9-8f5c-d46d6d62a49e.html)</li></ul>  |
| **Author**               | E.M. Anhaus (orignally from Atomic Blue Detections, Dan Beavin), oscd.community |


## Detection Rules

### Sigma rule

```
title: HH.exe Execution
id: 68c8acb4-1b60-4890-8e82-3ddf7a6dba84
description: Identifies usage of hh.exe executing recently modified .chm files.
status: experimental
author: E.M. Anhaus (orignally from Atomic Blue Detections, Dan Beavin), oscd.community
references:
    - https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1223/T1223.yaml
    - https://eqllib.readthedocs.io/en/latest/analytics/b25aa548-7937-11e9-8f5c-d46d6d62a49e.html
date: 2019/10/24
modified: 2019/11/11
tags:
    - attack.defense_evasion
    - attack.t1218.001
    - attack.execution  # an old one
    - attack.t1223  # an old one
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\hh.exe'
        CommandLine|contains: '.chm'
    condition: selection
fields:
    - ComputerName
    - User
    - CommandLine
falsepositives:
    - unlike
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\\\hh.exe" -and $_.message -match "CommandLine.*.*.chm.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\\\hh.exe AND winlog.event_data.CommandLine.keyword:*.chm*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/68c8acb4-1b60-4890-8e82-3ddf7a6dba84 <<EOF\n{\n  "metadata": {\n    "title": "HH.exe Execution",\n    "description": "Identifies usage of hh.exe executing recently modified .chm files.",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1218.001",\n      "attack.execution",\n      "attack.t1223"\n    ],\n    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\hh.exe AND winlog.event_data.CommandLine.keyword:*.chm*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\hh.exe AND winlog.event_data.CommandLine.keyword:*.chm*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'HH.exe Execution\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\nComputerName = {{_source.ComputerName}}\\n        User = {{_source.User}}\\n CommandLine = {{_source.CommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:*\\\\hh.exe AND CommandLine.keyword:*.chm*)
```


### splunk
    
```
(Image="*\\\\hh.exe" CommandLine="*.chm*") | table ComputerName,User,CommandLine
```


### logpoint
    
```
(Image="*\\\\hh.exe" CommandLine="*.chm*")
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\hh\\.exe)(?=.*.*\\.chm.*))'
```



