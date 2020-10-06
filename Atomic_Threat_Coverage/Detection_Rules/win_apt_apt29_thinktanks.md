| Title                    | APT29       |
|:-------------------------|:------------------|
| **Description**          | This method detects a suspicious powershell command line combination as used by APT29 in a campaign against US think tanks |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li><li>[T1059: Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059)</li><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           | <ul><li>[https://cloudblogs.microsoft.com/microsoftsecure/2018/12/03/analysis-of-cyberattack-on-u-s-think-tanks-non-profits-public-sector-by-unidentified-attackers/](https://cloudblogs.microsoft.com/microsoftsecure/2018/12/03/analysis-of-cyberattack-on-u-s-think-tanks-non-profits-public-sector-by-unidentified-attackers/)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>attack.g0016</li></ul> | 

## Detection Rules

### Sigma rule

```
title: APT29
id: 033fe7d6-66d1-4240-ac6b-28908009c71f
description: This method detects a suspicious powershell command line combination as used by APT29 in a campaign against US think tanks
references:
    - https://cloudblogs.microsoft.com/microsoftsecure/2018/12/03/analysis-of-cyberattack-on-u-s-think-tanks-non-profits-public-sector-by-unidentified-attackers/
tags:
    - attack.execution
    - attack.g0016
    - attack.t1086 # an old one
    - attack.t1059 # an old one
    - attack.t1059.001
author: Florian Roth
date: 2018/12/04
modified: 2020/08/26
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine: '*-noni -ep bypass $*'
    condition: selection
falsepositives:
    - unknown
level: critical

```





### powershell
    
```
Get-WinEvent | where {$_.message -match "CommandLine.*.*-noni -ep bypass $.*" } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:*\\-noni\\ \\-ep\\ bypass\\ $*
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/033fe7d6-66d1-4240-ac6b-28908009c71f <<EOF\n{\n  "metadata": {\n    "title": "APT29",\n    "description": "This method detects a suspicious powershell command line combination as used by APT29 in a campaign against US think tanks",\n    "tags": [\n      "attack.execution",\n      "attack.g0016",\n      "attack.t1086",\n      "attack.t1059",\n      "attack.t1059.001"\n    ],\n    "query": "winlog.event_data.CommandLine.keyword:*\\\\-noni\\\\ \\\\-ep\\\\ bypass\\\\ $*"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "winlog.event_data.CommandLine.keyword:*\\\\-noni\\\\ \\\\-ep\\\\ bypass\\\\ $*",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'APT29\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
CommandLine.keyword:*\\-noni \\-ep bypass $*
```


### splunk
    
```
CommandLine="*-noni -ep bypass $*"
```


### logpoint
    
```
CommandLine="*-noni -ep bypass $*"
```


### grep
    
```
grep -P '^.*-noni -ep bypass \\$.*'
```



