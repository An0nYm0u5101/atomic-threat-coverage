| Title                    | Shadow Copies Access via Symlink       |
|:-------------------------|:------------------|
| **Description**          | Shadow Copies storage symbolic link creation using operating systems utilities |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li><li>[T1003.002: Security Account Manager](https://attack.mitre.org/techniques/T1003/002)</li><li>[T1003.003: NTDS](https://attack.mitre.org/techniques/T1003/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li><li>[T1003.002: Security Account Manager](../Triggers/T1003.002.md)</li><li>[T1003.003: NTDS](../Triggers/T1003.003.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Legitimate administrator working with shadow copies, access for backup purposes</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment](https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment)</li></ul>  |
| **Author**               | Teymur Kheirkhabarov, oscd.community |


## Detection Rules

### Sigma rule

```
title: Shadow Copies Access via Symlink
id: 40b19fa6-d835-400c-b301-41f3a2baacaf
description: Shadow Copies storage symbolic link creation using operating systems utilities
author: Teymur Kheirkhabarov, oscd.community
date: 2019/10/22
references:
    - https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment
tags:
    - attack.credential_access
    - attack.t1003 # an old one
    - attack.t1003.002
    - attack.t1003.003
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine|contains|all:
            - mklink
            - HarddiskVolumeShadowCopy
    condition: selection
falsepositives:
    - Legitimate administrator working with shadow copies, access for backup purposes
status: experimental
level: medium

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "CommandLine.*.*mklink.*" -and $_.message -match "CommandLine.*.*HarddiskVolumeShadowCopy.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.CommandLine.keyword:*mklink* AND winlog.event_data.CommandLine.keyword:*HarddiskVolumeShadowCopy*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/40b19fa6-d835-400c-b301-41f3a2baacaf <<EOF\n{\n  "metadata": {\n    "title": "Shadow Copies Access via Symlink",\n    "description": "Shadow Copies storage symbolic link creation using operating systems utilities",\n    "tags": [\n      "attack.credential_access",\n      "attack.t1003",\n      "attack.t1003.002",\n      "attack.t1003.003"\n    ],\n    "query": "(winlog.event_data.CommandLine.keyword:*mklink* AND winlog.event_data.CommandLine.keyword:*HarddiskVolumeShadowCopy*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.CommandLine.keyword:*mklink* AND winlog.event_data.CommandLine.keyword:*HarddiskVolumeShadowCopy*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Shadow Copies Access via Symlink\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(CommandLine.keyword:*mklink* AND CommandLine.keyword:*HarddiskVolumeShadowCopy*)
```


### splunk
    
```
(CommandLine="*mklink*" CommandLine="*HarddiskVolumeShadowCopy*")
```


### logpoint
    
```
(CommandLine="*mklink*" CommandLine="*HarddiskVolumeShadowCopy*")
```


### grep
    
```
grep -P '^(?:.*(?=.*.*mklink.*)(?=.*.*HarddiskVolumeShadowCopy.*))'
```



