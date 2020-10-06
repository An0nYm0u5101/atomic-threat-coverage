| Title                    | MMC20 Lateral Movement       |
|:-------------------------|:------------------|
| **Description**          | Detects MMC20.Application Lateral Movement; specifically looks for the spawning of the parent MMC.exe with a command line of "-Embedding" as a child of svchost.exe |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1175: Component Object Model and Distributed COM](https://attack.mitre.org/techniques/T1175)</li><li>[T1021.003: Distributed Component Object Model](https://attack.mitre.org/techniques/T1021/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1021.003: Distributed Component Object Model](../Triggers/T1021.003.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unlikely</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           | <ul><li>[https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)</li><li>[https://drive.google.com/file/d/1lKya3_mLnR3UQuCoiYruO3qgu052_iS_/view?usp=sharing](https://drive.google.com/file/d/1lKya3_mLnR3UQuCoiYruO3qgu052_iS_/view?usp=sharing)</li></ul>  |
| **Author**               | @2xxeformyshirt (Security Risk Advisors) - rule; Teymur Kheirkhabarov (idea) |


## Detection Rules

### Sigma rule

```
title: MMC20 Lateral Movement
id: f1f3bf22-deb2-418d-8cce-e1a45e46a5bd
description: Detects MMC20.Application Lateral Movement; specifically looks for the spawning of the parent MMC.exe with a command line of "-Embedding" as a child of svchost.exe
author: '@2xxeformyshirt (Security Risk Advisors) - rule; Teymur Kheirkhabarov (idea)'
date: 2020/03/04
modified: 2020/08/23
references:
    - https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/
    - https://drive.google.com/file/d/1lKya3_mLnR3UQuCoiYruO3qgu052_iS_/view?usp=sharing
tags:
    - attack.execution
    - attack.t1175          # an old one
    - attack.t1021.003
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        ParentImage: '*\svchost.exe'
        Image: '*\mmc.exe'
        CommandLine: '*-Embedding*'
    condition: selection
falsepositives:
    - Unlikely
level: high

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "ParentImage.*.*\\\\svchost.exe" -and $_.message -match "Image.*.*\\\\mmc.exe" -and $_.message -match "CommandLine.*.*-Embedding.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.ParentImage.keyword:*\\\\svchost.exe AND winlog.event_data.Image.keyword:*\\\\mmc.exe AND winlog.event_data.CommandLine.keyword:*\\-Embedding*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/f1f3bf22-deb2-418d-8cce-e1a45e46a5bd <<EOF\n{\n  "metadata": {\n    "title": "MMC20 Lateral Movement",\n    "description": "Detects MMC20.Application Lateral Movement; specifically looks for the spawning of the parent MMC.exe with a command line of \\"-Embedding\\" as a child of svchost.exe",\n    "tags": [\n      "attack.execution",\n      "attack.t1175",\n      "attack.t1021.003"\n    ],\n    "query": "(winlog.event_data.ParentImage.keyword:*\\\\\\\\svchost.exe AND winlog.event_data.Image.keyword:*\\\\\\\\mmc.exe AND winlog.event_data.CommandLine.keyword:*\\\\-Embedding*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.ParentImage.keyword:*\\\\\\\\svchost.exe AND winlog.event_data.Image.keyword:*\\\\\\\\mmc.exe AND winlog.event_data.CommandLine.keyword:*\\\\-Embedding*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'MMC20 Lateral Movement\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(ParentImage.keyword:*\\\\svchost.exe AND Image.keyword:*\\\\mmc.exe AND CommandLine.keyword:*\\-Embedding*)
```


### splunk
    
```
(ParentImage="*\\\\svchost.exe" Image="*\\\\mmc.exe" CommandLine="*-Embedding*")
```


### logpoint
    
```
(ParentImage="*\\\\svchost.exe" Image="*\\\\mmc.exe" CommandLine="*-Embedding*")
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\svchost\\.exe)(?=.*.*\\mmc\\.exe)(?=.*.*-Embedding.*))'
```



