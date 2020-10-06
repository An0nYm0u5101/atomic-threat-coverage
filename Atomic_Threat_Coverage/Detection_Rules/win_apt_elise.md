| Title                    | Elise Backdoor       |
|:-------------------------|:------------------|
| **Description**          | Detects Elise backdoor acitivty as used by APT32 |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059: Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059)</li><li>[T1059.003: Windows Command Shell](https://attack.mitre.org/techniques/T1059/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.003: Windows Command Shell](../Triggers/T1059.003.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://community.rsa.com/community/products/netwitness/blog/2018/02/13/lotus-blossom-continues-asean-targeting](https://community.rsa.com/community/products/netwitness/blog/2018/02/13/lotus-blossom-continues-asean-targeting)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>attack.g0030</li><li>attack.g0050</li><li>attack.s0081</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Elise Backdoor
id: e507feb7-5f73-4ef6-a970-91bb6f6d744f
status: experimental
description: Detects Elise backdoor acitivty as used by APT32
references:
    - https://community.rsa.com/community/products/netwitness/blog/2018/02/13/lotus-blossom-continues-asean-targeting
tags:
    - attack.g0030
    - attack.g0050
    - attack.s0081
    - attack.execution
    - attack.t1059 # an old one
    - attack.t1059.003
author: Florian Roth
date: 2018/01/31
modified: 2020/08/26
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        Image: 'C:\Windows\SysWOW64\cmd.exe'
        CommandLine: '*\Windows\Caches\NavShExt.dll *'
    selection2:
        CommandLine: '*\AppData\Roaming\MICROS~1\Windows\Caches\NavShExt.dll,Setting'
    condition: 1 of them
falsepositives:
    - Unknown
level: critical

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "Image.*C:\\\\Windows\\\\SysWOW64\\\\cmd.exe" -and $_.message -match "CommandLine.*.*\\\\Windows\\\\Caches\\\\NavShExt.dll .*") -or $_.message -match "CommandLine.*.*\\\\AppData\\\\Roaming\\\\MICROS~1\\\\Windows\\\\Caches\\\\NavShExt.dll,Setting") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Image:"C\\:\\\\Windows\\\\SysWOW64\\\\cmd.exe" AND winlog.event_data.CommandLine.keyword:*\\\\Windows\\\\Caches\\\\NavShExt.dll\\ *) OR winlog.event_data.CommandLine.keyword:*\\\\AppData\\\\Roaming\\\\MICROS\\~1\\\\Windows\\\\Caches\\\\NavShExt.dll,Setting)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/e507feb7-5f73-4ef6-a970-91bb6f6d744f <<EOF\n{\n  "metadata": {\n    "title": "Elise Backdoor",\n    "description": "Detects Elise backdoor acitivty as used by APT32",\n    "tags": [\n      "attack.g0030",\n      "attack.g0050",\n      "attack.s0081",\n      "attack.execution",\n      "attack.t1059",\n      "attack.t1059.003"\n    ],\n    "query": "((winlog.event_data.Image:\\"C\\\\:\\\\\\\\Windows\\\\\\\\SysWOW64\\\\\\\\cmd.exe\\" AND winlog.event_data.CommandLine.keyword:*\\\\\\\\Windows\\\\\\\\Caches\\\\\\\\NavShExt.dll\\\\ *) OR winlog.event_data.CommandLine.keyword:*\\\\\\\\AppData\\\\\\\\Roaming\\\\\\\\MICROS\\\\~1\\\\\\\\Windows\\\\\\\\Caches\\\\\\\\NavShExt.dll,Setting)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.Image:\\"C\\\\:\\\\\\\\Windows\\\\\\\\SysWOW64\\\\\\\\cmd.exe\\" AND winlog.event_data.CommandLine.keyword:*\\\\\\\\Windows\\\\\\\\Caches\\\\\\\\NavShExt.dll\\\\ *) OR winlog.event_data.CommandLine.keyword:*\\\\\\\\AppData\\\\\\\\Roaming\\\\\\\\MICROS\\\\~1\\\\\\\\Windows\\\\\\\\Caches\\\\\\\\NavShExt.dll,Setting)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Elise Backdoor\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((Image:"C\\:\\\\Windows\\\\SysWOW64\\\\cmd.exe" AND CommandLine.keyword:*\\\\Windows\\\\Caches\\\\NavShExt.dll *) OR CommandLine.keyword:*\\\\AppData\\\\Roaming\\\\MICROS\\~1\\\\Windows\\\\Caches\\\\NavShExt.dll,Setting)
```


### splunk
    
```
((Image="C:\\\\Windows\\\\SysWOW64\\\\cmd.exe" CommandLine="*\\\\Windows\\\\Caches\\\\NavShExt.dll *") OR CommandLine="*\\\\AppData\\\\Roaming\\\\MICROS~1\\\\Windows\\\\Caches\\\\NavShExt.dll,Setting")
```


### logpoint
    
```
((Image="C:\\\\Windows\\\\SysWOW64\\\\cmd.exe" CommandLine="*\\\\Windows\\\\Caches\\\\NavShExt.dll *") OR CommandLine="*\\\\AppData\\\\Roaming\\\\MICROS~1\\\\Windows\\\\Caches\\\\NavShExt.dll,Setting")
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?=.*C:\\Windows\\SysWOW64\\cmd\\.exe)(?=.*.*\\Windows\\Caches\\NavShExt\\.dll .*))|.*.*\\AppData\\Roaming\\MICROS~1\\Windows\\Caches\\NavShExt\\.dll,Setting))'
```



