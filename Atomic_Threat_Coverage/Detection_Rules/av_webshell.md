| Title                    | Antivirus Web Shell Detection       |
|:-------------------------|:------------------|
| **Description**          | Detects a highly relevant Antivirus alert that reports a web shell |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0003: Persistence](https://attack.mitre.org/tactics/TA0003)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1100: Web Shell](https://attack.mitre.org/techniques/T1100)</li><li>[T1505.003: Web Shell](https://attack.mitre.org/techniques/T1505/003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0084_av_alert](../Data_Needed/DN_0084_av_alert.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1505.003: Web Shell](../Triggers/T1505.003.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Unlikely</li></ul>  |
| **Development Status**   |  Development Status wasn't defined for this Detection Rule yet  |
| **References**           | <ul><li>[https://www.nextron-systems.com/2018/09/08/antivirus-event-analysis-cheat-sheet-v1-4/](https://www.nextron-systems.com/2018/09/08/antivirus-event-analysis-cheat-sheet-v1-4/)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Antivirus Web Shell Detection
id: fdf135a2-9241-4f96-a114-bb404948f736
description: Detects a highly relevant Antivirus alert that reports a web shell
date: 2018/09/09
modified: 2019/10/04
author: Florian Roth
references:
    - https://www.nextron-systems.com/2018/09/08/antivirus-event-analysis-cheat-sheet-v1-4/
tags:
    - attack.persistence
    - attack.t1100
    - attack.t1505.003
logsource:
    product: antivirus
detection:
    selection:
        Signature:
            - "PHP/Backdoor*"
            - "JSP/Backdoor*"
            - "ASP/Backdoor*"
            - "Backdoor.PHP*"
            - "Backdoor.JSP*"
            - "Backdoor.ASP*"
            - "*Webshell*"
    condition: selection
fields:
    - FileName
    - User
falsepositives:
    - Unlikely
level: critical

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Signature.*PHP/Backdoor.*" -or $_.message -match "Signature.*JSP/Backdoor.*" -or $_.message -match "Signature.*ASP/Backdoor.*" -or $_.message -match "Signature.*Backdoor.PHP.*" -or $_.message -match "Signature.*Backdoor.JSP.*" -or $_.message -match "Signature.*Backdoor.ASP.*" -or $_.message -match "Signature.*.*Webshell.*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.Signature.keyword:(PHP\\/Backdoor* OR JSP\\/Backdoor* OR ASP\\/Backdoor* OR Backdoor.PHP* OR Backdoor.JSP* OR Backdoor.ASP* OR *Webshell*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/fdf135a2-9241-4f96-a114-bb404948f736 <<EOF\n{\n  "metadata": {\n    "title": "Antivirus Web Shell Detection",\n    "description": "Detects a highly relevant Antivirus alert that reports a web shell",\n    "tags": [\n      "attack.persistence",\n      "attack.t1100",\n      "attack.t1505.003"\n    ],\n    "query": "winlog.event_data.Signature.keyword:(PHP\\\\/Backdoor* OR JSP\\\\/Backdoor* OR ASP\\\\/Backdoor* OR Backdoor.PHP* OR Backdoor.JSP* OR Backdoor.ASP* OR *Webshell*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "winlog.event_data.Signature.keyword:(PHP\\\\/Backdoor* OR JSP\\\\/Backdoor* OR ASP\\\\/Backdoor* OR Backdoor.PHP* OR Backdoor.JSP* OR Backdoor.ASP* OR *Webshell*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Antivirus Web Shell Detection\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\nFileName = {{_source.FileName}}\\n    User = {{_source.User}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
Signature.keyword:(PHP\\/Backdoor* JSP\\/Backdoor* ASP\\/Backdoor* Backdoor.PHP* Backdoor.JSP* Backdoor.ASP* *Webshell*)
```


### splunk
    
```
(Signature="PHP/Backdoor*" OR Signature="JSP/Backdoor*" OR Signature="ASP/Backdoor*" OR Signature="Backdoor.PHP*" OR Signature="Backdoor.JSP*" OR Signature="Backdoor.ASP*" OR Signature="*Webshell*") | table FileName,User
```


### logpoint
    
```
Signature IN ["PHP/Backdoor*", "JSP/Backdoor*", "ASP/Backdoor*", "Backdoor.PHP*", "Backdoor.JSP*", "Backdoor.ASP*", "*Webshell*"]
```


### grep
    
```
grep -P '^(?:.*PHP/Backdoor.*|.*JSP/Backdoor.*|.*ASP/Backdoor.*|.*Backdoor\\.PHP.*|.*Backdoor\\.JSP.*|.*Backdoor\\.ASP.*|.*.*Webshell.*)'
```



