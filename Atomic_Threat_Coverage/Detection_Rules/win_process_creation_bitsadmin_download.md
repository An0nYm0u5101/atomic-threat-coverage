| Title                    | Bitsadmin Download       |
|:-------------------------|:------------------|
| **Description**          | Detects usage of bitsadmin downloading a file |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1197: BITS Jobs](https://attack.mitre.org/techniques/T1197)</li><li>[T1036.003: Rename System Utilities](https://attack.mitre.org/techniques/T1036/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1197: BITS Jobs](../Triggers/T1197.md)</li><li>[T1036.003: Rename System Utilities](../Triggers/T1036.003.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Some legitimate apps use this, but limited.</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://blog.netspi.com/15-ways-to-download-a-file/#bitsadmin](https://blog.netspi.com/15-ways-to-download-a-file/#bitsadmin)</li><li>[https://isc.sans.edu/diary/22264](https://isc.sans.edu/diary/22264)</li></ul>  |
| **Author**               | Michael Haag |
| Other Tags           | <ul><li>attack.s0190</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Bitsadmin Download
id: d059842b-6b9d-4ed1-b5c3-5b89143c6ede
status: experimental
description: Detects usage of bitsadmin downloading a file
references:
    - https://blog.netspi.com/15-ways-to-download-a-file/#bitsadmin
    - https://isc.sans.edu/diary/22264
tags:
    - attack.defense_evasion
    - attack.persistence
    - attack.t1197
    - attack.s0190
    - attack.t1036.003    
date: 2017/03/09
modified: 2020/09/06
author: Michael Haag
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        Image:
            - '*\bitsadmin.exe'
        CommandLine:
            - '* /transfer *'
    selection2:
        CommandLine:
            - '*copy bitsadmin.exe*'
    condition: selection1 or selection2
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Some legitimate apps use this, but limited.
level: medium

```





### powershell
    
```
Get-WinEvent | where {((($_.message -match "Image.*.*\\\\bitsadmin.exe") -and ($_.message -match "CommandLine.*.* /transfer .*")) -or ($_.message -match "CommandLine.*.*copy bitsadmin.exe.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Image.keyword:(*\\\\bitsadmin.exe) AND winlog.event_data.CommandLine.keyword:(*\\ \\/transfer\\ *)) OR winlog.event_data.CommandLine.keyword:(*copy\\ bitsadmin.exe*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/d059842b-6b9d-4ed1-b5c3-5b89143c6ede <<EOF\n{\n  "metadata": {\n    "title": "Bitsadmin Download",\n    "description": "Detects usage of bitsadmin downloading a file",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.persistence",\n      "attack.t1197",\n      "attack.s0190",\n      "attack.t1036.003"\n    ],\n    "query": "((winlog.event_data.Image.keyword:(*\\\\\\\\bitsadmin.exe) AND winlog.event_data.CommandLine.keyword:(*\\\\ \\\\/transfer\\\\ *)) OR winlog.event_data.CommandLine.keyword:(*copy\\\\ bitsadmin.exe*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.Image.keyword:(*\\\\\\\\bitsadmin.exe) AND winlog.event_data.CommandLine.keyword:(*\\\\ \\\\/transfer\\\\ *)) OR winlog.event_data.CommandLine.keyword:(*copy\\\\ bitsadmin.exe*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Bitsadmin Download\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((Image.keyword:(*\\\\bitsadmin.exe) AND CommandLine.keyword:(* \\/transfer *)) OR CommandLine.keyword:(*copy bitsadmin.exe*))
```


### splunk
    
```
(((Image="*\\\\bitsadmin.exe") (CommandLine="* /transfer *")) OR (CommandLine="*copy bitsadmin.exe*")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
((Image IN ["*\\\\bitsadmin.exe"] CommandLine IN ["* /transfer *"]) OR CommandLine IN ["*copy bitsadmin.exe*"])
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?=.*(?:.*.*\\bitsadmin\\.exe))(?=.*(?:.*.* /transfer .*)))|.*(?:.*.*copy bitsadmin\\.exe.*)))'
```



