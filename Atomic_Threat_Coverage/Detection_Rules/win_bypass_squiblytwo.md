| Title                    | SquiblyTwo       |
|:-------------------------|:------------------|
| **Description**          | Detects WMI SquiblyTwo Attack with possible renamed WMI by looking for imphash |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1047: Windows Management Instrumentation](https://attack.mitre.org/techniques/T1047)</li><li>[T1220: XSL Script Processing](https://attack.mitre.org/techniques/T1220)</li><li>[T1059.005: Visual Basic](https://attack.mitre.org/techniques/T1059/005)</li><li>[T1059.007: JavaScript/JScript](https://attack.mitre.org/techniques/T1059/007)</li><li>[T1059: Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1047: Windows Management Instrumentation](../Triggers/T1047.md)</li><li>[T1220: XSL Script Processing](../Triggers/T1220.md)</li><li>[T1059.005: Visual Basic](../Triggers/T1059.005.md)</li></ul>  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://subt0x11.blogspot.ch/2018/04/wmicexe-whitelisting-bypass-hacking.html](https://subt0x11.blogspot.ch/2018/04/wmicexe-whitelisting-bypass-hacking.html)</li><li>[https://twitter.com/mattifestation/status/986280382042595328](https://twitter.com/mattifestation/status/986280382042595328)</li></ul>  |
| **Author**               | Markus Neis / Florian Roth |


## Detection Rules

### Sigma rule

```
title: SquiblyTwo
id: 8d63dadf-b91b-4187-87b6-34a1114577ea
status: experimental
description: Detects WMI SquiblyTwo Attack with possible renamed WMI by looking for imphash
references:
    - https://subt0x11.blogspot.ch/2018/04/wmicexe-whitelisting-bypass-hacking.html
    - https://twitter.com/mattifestation/status/986280382042595328
tags:
    - attack.defense_evasion
    - attack.t1047
    - attack.t1220
    - attack.execution
    - attack.t1059.005
    - attack.t1059.007
    - attack.t1059  # an old one
author: Markus Neis / Florian Roth
date: 2019/01/16
modified: 2020/08/27
falsepositives:
    - Unknown
level: medium
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        Image:
            - '*\wmic.exe'
        CommandLine:
            - wmic * *format:\"http*
            - wmic * /format:'http
            - wmic * /format:http*
    selection2:
        Imphash:
            - 1B1A3F43BF37B5BFE60751F2EE2F326E
            - 37777A96245A3C74EB217308F3546F4C
            - 9D87C9D67CE724033C0B40CC4CA1B206
        CommandLine:
            - '* *format:\"http*'
            - '* /format:''http'
            - '* /format:http*'
    condition: 1 of them

```





### powershell
    
```
Get-WinEvent | where {((($_.message -match "Image.*.*\\\\wmic.exe") -and ($_.message -match "CommandLine.*wmic .* .*format:\\\\\\"http.*" -or $_.message -match "CommandLine.*wmic .* /format:\'http" -or $_.message -match "CommandLine.*wmic .* /format:http.*")) -or (($_.message -match "1B1A3F43BF37B5BFE60751F2EE2F326E" -or $_.message -match "37777A96245A3C74EB217308F3546F4C" -or $_.message -match "9D87C9D67CE724033C0B40CC4CA1B206") -and ($_.message -match "CommandLine.*.* .*format:\\\\\\"http.*" -or $_.message -match "CommandLine.*.* /format:\'http" -or $_.message -match "CommandLine.*.* /format:http.*"))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.Image.keyword:(*\\\\wmic.exe) AND winlog.event_data.CommandLine.keyword:(wmic\\ *\\ *format\\:\\\\\\"http* OR wmic\\ *\\ \\/format\\:\'http OR wmic\\ *\\ \\/format\\:http*)) OR (winlog.event_data.Imphash:("1b1a3f43bf37b5bfe60751f2ee2f326e" OR "1B1A3F43BF37B5BFE60751F2EE2F326E" OR "37777a96245a3c74eb217308f3546f4c" OR "37777A96245A3C74EB217308F3546F4C" OR "9d87c9d67ce724033c0b40cc4ca1b206" OR "9D87C9D67CE724033C0B40CC4CA1B206") AND winlog.event_data.CommandLine.keyword:(*\\ *format\\:\\\\\\"http* OR *\\ \\/format\\:\'http OR *\\ \\/format\\:http*)))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/8d63dadf-b91b-4187-87b6-34a1114577ea <<EOF\n{\n  "metadata": {\n    "title": "SquiblyTwo",\n    "description": "Detects WMI SquiblyTwo Attack with possible renamed WMI by looking for imphash",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1047",\n      "attack.t1220",\n      "attack.execution",\n      "attack.t1059.005",\n      "attack.t1059.007",\n      "attack.t1059"\n    ],\n    "query": "((winlog.event_data.Image.keyword:(*\\\\\\\\wmic.exe) AND winlog.event_data.CommandLine.keyword:(wmic\\\\ *\\\\ *format\\\\:\\\\\\\\\\\\\\"http* OR wmic\\\\ *\\\\ \\\\/format\\\\:\'http OR wmic\\\\ *\\\\ \\\\/format\\\\:http*)) OR (winlog.event_data.Imphash:(\\"1b1a3f43bf37b5bfe60751f2ee2f326e\\" OR \\"1B1A3F43BF37B5BFE60751F2EE2F326E\\" OR \\"37777a96245a3c74eb217308f3546f4c\\" OR \\"37777A96245A3C74EB217308F3546F4C\\" OR \\"9d87c9d67ce724033c0b40cc4ca1b206\\" OR \\"9D87C9D67CE724033C0B40CC4CA1B206\\") AND winlog.event_data.CommandLine.keyword:(*\\\\ *format\\\\:\\\\\\\\\\\\\\"http* OR *\\\\ \\\\/format\\\\:\'http OR *\\\\ \\\\/format\\\\:http*)))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.Image.keyword:(*\\\\\\\\wmic.exe) AND winlog.event_data.CommandLine.keyword:(wmic\\\\ *\\\\ *format\\\\:\\\\\\\\\\\\\\"http* OR wmic\\\\ *\\\\ \\\\/format\\\\:\'http OR wmic\\\\ *\\\\ \\\\/format\\\\:http*)) OR (winlog.event_data.Imphash:(\\"1b1a3f43bf37b5bfe60751f2ee2f326e\\" OR \\"1B1A3F43BF37B5BFE60751F2EE2F326E\\" OR \\"37777a96245a3c74eb217308f3546f4c\\" OR \\"37777A96245A3C74EB217308F3546F4C\\" OR \\"9d87c9d67ce724033c0b40cc4ca1b206\\" OR \\"9D87C9D67CE724033C0B40CC4CA1B206\\") AND winlog.event_data.CommandLine.keyword:(*\\\\ *format\\\\:\\\\\\\\\\\\\\"http* OR *\\\\ \\\\/format\\\\:\'http OR *\\\\ \\\\/format\\\\:http*)))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'SquiblyTwo\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((Image.keyword:(*\\\\wmic.exe) AND CommandLine.keyword:(wmic * *format\\:\\\\\\"http* wmic * \\/format\\:\'http wmic * \\/format\\:http*)) OR (Imphash:("1b1a3f43bf37b5bfe60751f2ee2f326e" "1B1A3F43BF37B5BFE60751F2EE2F326E" "37777a96245a3c74eb217308f3546f4c" "37777A96245A3C74EB217308F3546F4C" "9d87c9d67ce724033c0b40cc4ca1b206" "9D87C9D67CE724033C0B40CC4CA1B206") AND CommandLine.keyword:(* *format\\:\\\\\\"http* * \\/format\\:\'http * \\/format\\:http*)))
```


### splunk
    
```
(((Image="*\\\\wmic.exe") (CommandLine="wmic * *format:\\\\\\"http*" OR CommandLine="wmic * /format:\'http" OR CommandLine="wmic * /format:http*")) OR ((Imphash="1B1A3F43BF37B5BFE60751F2EE2F326E" OR Imphash="37777A96245A3C74EB217308F3546F4C" OR Imphash="9D87C9D67CE724033C0B40CC4CA1B206") (CommandLine="* *format:\\\\\\"http*" OR CommandLine="* /format:\'http" OR CommandLine="* /format:http*")))
```


### logpoint
    
```
((Image IN ["*\\\\wmic.exe"] CommandLine IN ["wmic * *format:\\\\\\"http*", "wmic * /format:\'http", "wmic * /format:http*"]) OR (Imphash IN ["1B1A3F43BF37B5BFE60751F2EE2F326E", "37777A96245A3C74EB217308F3546F4C", "9D87C9D67CE724033C0B40CC4CA1B206"] CommandLine IN ["* *format:\\\\\\"http*", "* /format:\'http", "* /format:http*"]))
```


### grep
    
```
grep -P \'^(?:.*(?:.*(?:.*(?=.*(?:.*.*\\wmic\\.exe))(?=.*(?:.*wmic .* .*format:\\"http.*|.*wmic .* /format:\'"\'"\'http|.*wmic .* /format:http.*)))|.*(?:.*(?=.*(?:.*1B1A3F43BF37B5BFE60751F2EE2F326E|.*37777A96245A3C74EB217308F3546F4C|.*9D87C9D67CE724033C0B40CC4CA1B206))(?=.*(?:.*.* .*format:\\"http.*|.*.* /format:\'"\'"\'http|.*.* /format:http.*)))))\'
```



