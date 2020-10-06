| Title                    | Empire PowerShell Launch Parameters       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious powershell command line parameters used in Empire |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      |  There are no documented False Positives for this Detection Rule yet  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://github.com/EmpireProject/Empire/blob/c2ba61ca8d2031dad0cfc1d5770ba723e8b710db/lib/common/helpers.py#L165](https://github.com/EmpireProject/Empire/blob/c2ba61ca8d2031dad0cfc1d5770ba723e8b710db/lib/common/helpers.py#L165)</li><li>[https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/modules/powershell/persistence/powerbreach/deaduser.py#L191](https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/modules/powershell/persistence/powerbreach/deaduser.py#L191)</li><li>[https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/modules/powershell/persistence/powerbreach/resolver.py#L178](https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/modules/powershell/persistence/powerbreach/resolver.py#L178)</li><li>[https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/data/module_source/privesc/Invoke-EventVwrBypass.ps1#L64](https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/data/module_source/privesc/Invoke-EventVwrBypass.ps1#L64)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Empire PowerShell Launch Parameters
id: 79f4ede3-402e-41c8-bc3e-ebbf5f162581
description: Detects suspicious powershell command line parameters used in Empire
status: experimental
references:
    - https://github.com/EmpireProject/Empire/blob/c2ba61ca8d2031dad0cfc1d5770ba723e8b710db/lib/common/helpers.py#L165
    - https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/modules/powershell/persistence/powerbreach/deaduser.py#L191
    - https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/lib/modules/powershell/persistence/powerbreach/resolver.py#L178
    - https://github.com/EmpireProject/Empire/blob/e37fb2eef8ff8f5a0a689f1589f424906fe13055/data/module_source/privesc/Invoke-EventVwrBypass.ps1#L64
author: Florian Roth
date: 2019/04/20
modified: 2020/07/13
tags:
    - attack.execution
    - attack.t1059.001
    - attack.t1086          # an old one
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine|contains:
            - ' -NoP -sta -NonI -W Hidden -Enc '
            - ' -noP -sta -w 1 -enc '
            - ' -NoP -NonI -W Hidden -enc '
            - ' -noP -sta -w 1 -enc'
            - ' -enc  SQB'
            - ' -nop -exec bypass -EncodedCommand SQB'
    condition: selection
level: critical

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "CommandLine.*.* -NoP -sta -NonI -W Hidden -Enc .*" -or $_.message -match "CommandLine.*.* -noP -sta -w 1 -enc .*" -or $_.message -match "CommandLine.*.* -NoP -NonI -W Hidden -enc .*" -or $_.message -match "CommandLine.*.* -noP -sta -w 1 -enc.*" -or $_.message -match "CommandLine.*.* -enc  SQB.*" -or $_.message -match "CommandLine.*.* -nop -exec bypass -EncodedCommand SQB.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:(*\\ \\-NoP\\ \\-sta\\ \\-NonI\\ \\-W\\ Hidden\\ \\-Enc\\ * OR *\\ \\-noP\\ \\-sta\\ \\-w\\ 1\\ \\-enc\\ * OR *\\ \\-NoP\\ \\-NonI\\ \\-W\\ Hidden\\ \\-enc\\ * OR *\\ \\-noP\\ \\-sta\\ \\-w\\ 1\\ \\-enc* OR *\\ \\-enc\\ \\ SQB* OR *\\ \\-nop\\ \\-exec\\ bypass\\ \\-EncodedCommand\\ SQB*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/79f4ede3-402e-41c8-bc3e-ebbf5f162581 <<EOF\n{\n  "metadata": {\n    "title": "Empire PowerShell Launch Parameters",\n    "description": "Detects suspicious powershell command line parameters used in Empire",\n    "tags": [\n      "attack.execution",\n      "attack.t1059.001",\n      "attack.t1086"\n    ],\n    "query": "winlog.event_data.CommandLine.keyword:(*\\\\ \\\\-NoP\\\\ \\\\-sta\\\\ \\\\-NonI\\\\ \\\\-W\\\\ Hidden\\\\ \\\\-Enc\\\\ * OR *\\\\ \\\\-noP\\\\ \\\\-sta\\\\ \\\\-w\\\\ 1\\\\ \\\\-enc\\\\ * OR *\\\\ \\\\-NoP\\\\ \\\\-NonI\\\\ \\\\-W\\\\ Hidden\\\\ \\\\-enc\\\\ * OR *\\\\ \\\\-noP\\\\ \\\\-sta\\\\ \\\\-w\\\\ 1\\\\ \\\\-enc* OR *\\\\ \\\\-enc\\\\ \\\\ SQB* OR *\\\\ \\\\-nop\\\\ \\\\-exec\\\\ bypass\\\\ \\\\-EncodedCommand\\\\ SQB*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "winlog.event_data.CommandLine.keyword:(*\\\\ \\\\-NoP\\\\ \\\\-sta\\\\ \\\\-NonI\\\\ \\\\-W\\\\ Hidden\\\\ \\\\-Enc\\\\ * OR *\\\\ \\\\-noP\\\\ \\\\-sta\\\\ \\\\-w\\\\ 1\\\\ \\\\-enc\\\\ * OR *\\\\ \\\\-NoP\\\\ \\\\-NonI\\\\ \\\\-W\\\\ Hidden\\\\ \\\\-enc\\\\ * OR *\\\\ \\\\-noP\\\\ \\\\-sta\\\\ \\\\-w\\\\ 1\\\\ \\\\-enc* OR *\\\\ \\\\-enc\\\\ \\\\ SQB* OR *\\\\ \\\\-nop\\\\ \\\\-exec\\\\ bypass\\\\ \\\\-EncodedCommand\\\\ SQB*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Empire PowerShell Launch Parameters\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
CommandLine.keyword:(* \\-NoP \\-sta \\-NonI \\-W Hidden \\-Enc * * \\-noP \\-sta \\-w 1 \\-enc * * \\-NoP \\-NonI \\-W Hidden \\-enc * * \\-noP \\-sta \\-w 1 \\-enc* * \\-enc  SQB* * \\-nop \\-exec bypass \\-EncodedCommand SQB*)
```


### splunk
    
```
(CommandLine="* -NoP -sta -NonI -W Hidden -Enc *" OR CommandLine="* -noP -sta -w 1 -enc *" OR CommandLine="* -NoP -NonI -W Hidden -enc *" OR CommandLine="* -noP -sta -w 1 -enc*" OR CommandLine="* -enc  SQB*" OR CommandLine="* -nop -exec bypass -EncodedCommand SQB*")
```


### logpoint
    
```
CommandLine IN ["* -NoP -sta -NonI -W Hidden -Enc *", "* -noP -sta -w 1 -enc *", "* -NoP -NonI -W Hidden -enc *", "* -noP -sta -w 1 -enc*", "* -enc  SQB*", "* -nop -exec bypass -EncodedCommand SQB*"]
```


### grep
    
```
grep -P '^(?:.*.* -NoP -sta -NonI -W Hidden -Enc .*|.*.* -noP -sta -w 1 -enc .*|.*.* -NoP -NonI -W Hidden -enc .*|.*.* -noP -sta -w 1 -enc.*|.*.* -enc  SQB.*|.*.* -nop -exec bypass -EncodedCommand SQB.*)'
```



