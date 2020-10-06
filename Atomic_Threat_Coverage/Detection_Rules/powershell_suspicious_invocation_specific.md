| Title                    | Suspicious PowerShell Invocations - Specific       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious PowerShell invocation command parameters |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li></ul>  |
| **Data Needed**          |  There is no documented Data Needed for this Detection Rule yet  |
| **Trigger**              | <ul><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Penetration tests</li></ul>  |
| **Development Status**   | experimental |
| **References**           |  There are no documented References for this Detection Rule yet  |
| **Author**               | Florian Roth (rule) |


## Detection Rules

### Sigma rule

```
title: Suspicious PowerShell Invocations - Specific
id: fce5f582-cc00-41e1-941a-c6fabf0fdb8c
status: experimental
description: Detects suspicious PowerShell invocation command parameters
tags:
    - attack.execution
    - attack.t1059.001
    - attack.t1086  #an old one
author: Florian Roth (rule)
date: 2017/03/05
logsource:
    product: windows
    service: powershell
detection:
    keywords:
        Message:
            - '* -nop -w hidden -c * [Convert]::FromBase64String*'
            - '* -w hidden -noni -nop -c "iex(New-Object*'
            - '* -w hidden -ep bypass -Enc*'
            - '*powershell.exe reg add HKCU\software\microsoft\windows\currentversion\run*'
            - '*bypass -noprofile -windowstyle hidden (new-object system.net.webclient).download*'
            - '*iex(New-Object Net.WebClient).Download*'
    condition: keywords
falsepositives:
    - Penetration tests
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-PowerShell/Operational | where {(($_.message -match ".* -nop -w hidden -c .* [Convert]::FromBase64String.*" -or $_.message -match ".* -w hidden -noni -nop -c \\"iex(New-Object.*" -or $_.message -match ".* -w hidden -ep bypass -Enc.*" -or $_.message -match ".*powershell.exe reg add HKCU\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\run.*" -or $_.message -match ".*bypass -noprofile -windowstyle hidden (new-object system.net.webclient).download.*" -or $_.message -match ".*iex(New-Object Net.WebClient).Download.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
Message.keyword:(*\\ \\-nop\\ \\-w\\ hidden\\ \\-c\\ *\\ \\[Convert\\]\\:\\:FromBase64String* OR *\\ \\-w\\ hidden\\ \\-noni\\ \\-nop\\ \\-c\\ \\"iex\\(New\\-Object* OR *\\ \\-w\\ hidden\\ \\-ep\\ bypass\\ \\-Enc* OR *powershell.exe\\ reg\\ add\\ HKCU\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\run* OR *bypass\\ \\-noprofile\\ \\-windowstyle\\ hidden\\ \\(new\\-object\\ system.net.webclient\\).download* OR *iex\\(New\\-Object\\ Net.WebClient\\).Download*)
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/fce5f582-cc00-41e1-941a-c6fabf0fdb8c <<EOF\n{\n  "metadata": {\n    "title": "Suspicious PowerShell Invocations - Specific",\n    "description": "Detects suspicious PowerShell invocation command parameters",\n    "tags": [\n      "attack.execution",\n      "attack.t1059.001",\n      "attack.t1086"\n    ],\n    "query": "Message.keyword:(*\\\\ \\\\-nop\\\\ \\\\-w\\\\ hidden\\\\ \\\\-c\\\\ *\\\\ \\\\[Convert\\\\]\\\\:\\\\:FromBase64String* OR *\\\\ \\\\-w\\\\ hidden\\\\ \\\\-noni\\\\ \\\\-nop\\\\ \\\\-c\\\\ \\\\\\"iex\\\\(New\\\\-Object* OR *\\\\ \\\\-w\\\\ hidden\\\\ \\\\-ep\\\\ bypass\\\\ \\\\-Enc* OR *powershell.exe\\\\ reg\\\\ add\\\\ HKCU\\\\\\\\software\\\\\\\\microsoft\\\\\\\\windows\\\\\\\\currentversion\\\\\\\\run* OR *bypass\\\\ \\\\-noprofile\\\\ \\\\-windowstyle\\\\ hidden\\\\ \\\\(new\\\\-object\\\\ system.net.webclient\\\\).download* OR *iex\\\\(New\\\\-Object\\\\ Net.WebClient\\\\).Download*)"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "Message.keyword:(*\\\\ \\\\-nop\\\\ \\\\-w\\\\ hidden\\\\ \\\\-c\\\\ *\\\\ \\\\[Convert\\\\]\\\\:\\\\:FromBase64String* OR *\\\\ \\\\-w\\\\ hidden\\\\ \\\\-noni\\\\ \\\\-nop\\\\ \\\\-c\\\\ \\\\\\"iex\\\\(New\\\\-Object* OR *\\\\ \\\\-w\\\\ hidden\\\\ \\\\-ep\\\\ bypass\\\\ \\\\-Enc* OR *powershell.exe\\\\ reg\\\\ add\\\\ HKCU\\\\\\\\software\\\\\\\\microsoft\\\\\\\\windows\\\\\\\\currentversion\\\\\\\\run* OR *bypass\\\\ \\\\-noprofile\\\\ \\\\-windowstyle\\\\ hidden\\\\ \\\\(new\\\\-object\\\\ system.net.webclient\\\\).download* OR *iex\\\\(New\\\\-Object\\\\ Net.WebClient\\\\).Download*)",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Suspicious PowerShell Invocations - Specific\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
Message.keyword:(* \\-nop \\-w hidden \\-c * \\[Convert\\]\\:\\:FromBase64String* * \\-w hidden \\-noni \\-nop \\-c \\"iex\\(New\\-Object* * \\-w hidden \\-ep bypass \\-Enc* *powershell.exe reg add HKCU\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\run* *bypass \\-noprofile \\-windowstyle hidden \\(new\\-object system.net.webclient\\).download* *iex\\(New\\-Object Net.WebClient\\).Download*)
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-PowerShell/Operational" (Message="* -nop -w hidden -c * [Convert]::FromBase64String*" OR Message="* -w hidden -noni -nop -c \\"iex(New-Object*" OR Message="* -w hidden -ep bypass -Enc*" OR Message="*powershell.exe reg add HKCU\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\run*" OR Message="*bypass -noprofile -windowstyle hidden (new-object system.net.webclient).download*" OR Message="*iex(New-Object Net.WebClient).Download*"))
```


### logpoint
    
```
Message IN ["* -nop -w hidden -c * [Convert]::FromBase64String*", "* -w hidden -noni -nop -c \\"iex(New-Object*", "* -w hidden -ep bypass -Enc*", "*powershell.exe reg add HKCU\\\\software\\\\microsoft\\\\windows\\\\currentversion\\\\run*", "*bypass -noprofile -windowstyle hidden (new-object system.net.webclient).download*", "*iex(New-Object Net.WebClient).Download*"]
```


### grep
    
```
grep -P \'^(?:.*.* -nop -w hidden -c .* \\[Convert\\]::FromBase64String.*|.*.* -w hidden -noni -nop -c "iex\\(New-Object.*|.*.* -w hidden -ep bypass -Enc.*|.*.*powershell\\.exe reg add HKCU\\software\\microsoft\\windows\\currentversion\\run.*|.*.*bypass -noprofile -windowstyle hidden \\(new-object system\\.net\\.webclient\\)\\.download.*|.*.*iex\\(New-Object Net\\.WebClient\\)\\.Download.*)\'
```



