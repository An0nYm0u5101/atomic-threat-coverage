| Title                    | Powershell AMSI Bypass via .NET Reflection       |
|:-------------------------|:------------------|
| **Description**          | Detects Request to amsiInitFailed that can be used to disable AMSI Scanning |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1089: Disabling Security Tools](https://attack.mitre.org/techniques/T1089)</li><li>[T1562.001: Disable or Modify Tools](https://attack.mitre.org/techniques/T1562/001)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1562.001: Disable or Modify Tools](../Triggers/T1562.001.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Potential Admin Activity</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://twitter.com/mattifestation/status/735261176745988096](https://twitter.com/mattifestation/status/735261176745988096)</li><li>[https://www.hybrid-analysis.com/sample/0ced17419e01663a0cd836c9c2eb925e3031ffb5b18ccf35f4dea5d586d0203e?environmentId=120](https://www.hybrid-analysis.com/sample/0ced17419e01663a0cd836c9c2eb925e3031ffb5b18ccf35f4dea5d586d0203e?environmentId=120)</li></ul>  |
| **Author**               | Markus Neis |


## Detection Rules

### Sigma rule

```
title: Powershell AMSI Bypass via .NET Reflection
id: 30edb182-aa75-42c0-b0a9-e998bb29067c
status: experimental
description: Detects Request to amsiInitFailed that can be used to disable AMSI Scanning
references:
    - https://twitter.com/mattifestation/status/735261176745988096
    - https://www.hybrid-analysis.com/sample/0ced17419e01663a0cd836c9c2eb925e3031ffb5b18ccf35f4dea5d586d0203e?environmentId=120
tags:
    - attack.defense_evasion
    - attack.t1089         # an old one
    - attack.t1562.001
author: Markus Neis
date: 2018/08/17
modified: 2020/09/01
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        CommandLine:
            - '*System.Management.Automation.AmsiUtils*'
    selection2:
        CommandLine:
            - '*amsiInitFailed*'
    condition: selection1 and selection2
falsepositives:
    - Potential Admin Activity
level: high

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "CommandLine.*.*System.Management.Automation.AmsiUtils.*") -and ($_.message -match "CommandLine.*.*amsiInitFailed.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.CommandLine.keyword:(*System.Management.Automation.AmsiUtils*) AND winlog.event_data.CommandLine.keyword:(*amsiInitFailed*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/30edb182-aa75-42c0-b0a9-e998bb29067c <<EOF\n{\n  "metadata": {\n    "title": "Powershell AMSI Bypass via .NET Reflection",\n    "description": "Detects Request to amsiInitFailed that can be used to disable AMSI Scanning",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1089",\n      "attack.t1562.001"\n    ],\n    "query": "(winlog.event_data.CommandLine.keyword:(*System.Management.Automation.AmsiUtils*) AND winlog.event_data.CommandLine.keyword:(*amsiInitFailed*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.CommandLine.keyword:(*System.Management.Automation.AmsiUtils*) AND winlog.event_data.CommandLine.keyword:(*amsiInitFailed*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Powershell AMSI Bypass via .NET Reflection\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(CommandLine.keyword:(*System.Management.Automation.AmsiUtils*) AND CommandLine.keyword:(*amsiInitFailed*))
```


### splunk
    
```
((CommandLine="*System.Management.Automation.AmsiUtils*") (CommandLine="*amsiInitFailed*"))
```


### logpoint
    
```
(CommandLine IN ["*System.Management.Automation.AmsiUtils*"] CommandLine IN ["*amsiInitFailed*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*(?:.*.*System\\.Management\\.Automation\\.AmsiUtils.*))(?=.*(?:.*.*amsiInitFailed.*)))'
```



