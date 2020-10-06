| Title                    | Suspicious Scripting in a WMI Consumer       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious scripting in WMI Event Consumers |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li><li>[T1059.005: Visual Basic](https://attack.mitre.org/techniques/T1059/005)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0023_20_windows_sysmon_WmiEvent](../Data_Needed/DN_0023_20_windows_sysmon_WmiEvent.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.005: Visual Basic](../Triggers/T1059.005.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Administrative scripts</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://in.security/an-intro-into-abusing-and-identifying-wmi-event-subscriptions-for-persistence/](https://in.security/an-intro-into-abusing-and-identifying-wmi-event-subscriptions-for-persistence/)</li><li>[https://github.com/Neo23x0/signature-base/blob/master/yara/gen_susp_lnk_files.yar#L19](https://github.com/Neo23x0/signature-base/blob/master/yara/gen_susp_lnk_files.yar#L19)</li></ul>  |
| **Author**               | Florian Roth |


## Detection Rules

### Sigma rule

```
title: Suspicious Scripting in a WMI Consumer
id: fe21810c-2a8c-478f-8dd3-5a287fb2a0e0
status: experimental
description: Detects suspicious scripting in WMI Event Consumers
author: Florian Roth
references:
    - https://in.security/an-intro-into-abusing-and-identifying-wmi-event-subscriptions-for-persistence/
    - https://github.com/Neo23x0/signature-base/blob/master/yara/gen_susp_lnk_files.yar#L19
date: 2019/04/15
tags:
    - attack.t1086          # an old one
    - attack.execution
    - attack.t1059.005
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 20
        Destination:
            - '*new-object system.net.webclient).downloadstring(*'
            - '*new-object system.net.webclient).downloadfile(*'
            - '*new-object net.webclient).downloadstring(*'
            - '*new-object net.webclient).downloadfile(*'
            - '* iex(*'
            - '*WScript.shell*'
            - '* -nop *'
            - '* -noprofile *'
            - '* -decode *'
            - '* -enc *'
    condition: selection
fields:
    - CommandLine
    - ParentCommandLine
falsepositives:
    - Administrative scripts
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "20" -and ($_.message -match "Destination.*.*new-object system.net.webclient).downloadstring(.*" -or $_.message -match "Destination.*.*new-object system.net.webclient).downloadfile(.*" -or $_.message -match "Destination.*.*new-object net.webclient).downloadstring(.*" -or $_.message -match "Destination.*.*new-object net.webclient).downloadfile(.*" -or $_.message -match "Destination.*.* iex(.*" -or $_.message -match "Destination.*.*WScript.shell.*" -or $_.message -match "Destination.*.* -nop .*" -or $_.message -match "Destination.*.* -noprofile .*" -or $_.message -match "Destination.*.* -decode .*" -or $_.message -match "Destination.*.* -enc .*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\\-Windows\\-Sysmon\\/Operational" AND winlog.event_id:"20" AND Destination.keyword:(*new\\-object\\ system.net.webclient\\).downloadstring\\(* OR *new\\-object\\ system.net.webclient\\).downloadfile\\(* OR *new\\-object\\ net.webclient\\).downloadstring\\(* OR *new\\-object\\ net.webclient\\).downloadfile\\(* OR *\\ iex\\(* OR *WScript.shell* OR *\\ \\-nop\\ * OR *\\ \\-noprofile\\ * OR *\\ \\-decode\\ * OR *\\ \\-enc\\ *))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/fe21810c-2a8c-478f-8dd3-5a287fb2a0e0 <<EOF\n{\n  "metadata": {\n    "title": "Suspicious Scripting in a WMI Consumer",\n    "description": "Detects suspicious scripting in WMI Event Consumers",\n    "tags": [\n      "attack.t1086",\n      "attack.execution",\n      "attack.t1059.005"\n    ],\n    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"20\\" AND Destination.keyword:(*new\\\\-object\\\\ system.net.webclient\\\\).downloadstring\\\\(* OR *new\\\\-object\\\\ system.net.webclient\\\\).downloadfile\\\\(* OR *new\\\\-object\\\\ net.webclient\\\\).downloadstring\\\\(* OR *new\\\\-object\\\\ net.webclient\\\\).downloadfile\\\\(* OR *\\\\ iex\\\\(* OR *WScript.shell* OR *\\\\ \\\\-nop\\\\ * OR *\\\\ \\\\-noprofile\\\\ * OR *\\\\ \\\\-decode\\\\ * OR *\\\\ \\\\-enc\\\\ *))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.channel:\\"Microsoft\\\\-Windows\\\\-Sysmon\\\\/Operational\\" AND winlog.event_id:\\"20\\" AND Destination.keyword:(*new\\\\-object\\\\ system.net.webclient\\\\).downloadstring\\\\(* OR *new\\\\-object\\\\ system.net.webclient\\\\).downloadfile\\\\(* OR *new\\\\-object\\\\ net.webclient\\\\).downloadstring\\\\(* OR *new\\\\-object\\\\ net.webclient\\\\).downloadfile\\\\(* OR *\\\\ iex\\\\(* OR *WScript.shell* OR *\\\\ \\\\-nop\\\\ * OR *\\\\ \\\\-noprofile\\\\ * OR *\\\\ \\\\-decode\\\\ * OR *\\\\ \\\\-enc\\\\ *))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Suspicious Scripting in a WMI Consumer\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\\n      CommandLine = {{_source.CommandLine}}\\nParentCommandLine = {{_source.ParentCommandLine}}================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(EventID:"20" AND Destination.keyword:(*new\\-object system.net.webclient\\).downloadstring\\(* *new\\-object system.net.webclient\\).downloadfile\\(* *new\\-object net.webclient\\).downloadstring\\(* *new\\-object net.webclient\\).downloadfile\\(* * iex\\(* *WScript.shell* * \\-nop * * \\-noprofile * * \\-decode * * \\-enc *))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="20" (Destination="*new-object system.net.webclient).downloadstring(*" OR Destination="*new-object system.net.webclient).downloadfile(*" OR Destination="*new-object net.webclient).downloadstring(*" OR Destination="*new-object net.webclient).downloadfile(*" OR Destination="* iex(*" OR Destination="*WScript.shell*" OR Destination="* -nop *" OR Destination="* -noprofile *" OR Destination="* -decode *" OR Destination="* -enc *")) | table CommandLine,ParentCommandLine
```


### logpoint
    
```
(event_id="20" Destination IN ["*new-object system.net.webclient).downloadstring(*", "*new-object system.net.webclient).downloadfile(*", "*new-object net.webclient).downloadstring(*", "*new-object net.webclient).downloadfile(*", "* iex(*", "*WScript.shell*", "* -nop *", "* -noprofile *", "* -decode *", "* -enc *"])
```


### grep
    
```
grep -P '^(?:.*(?=.*20)(?=.*(?:.*.*new-object system\\.net\\.webclient\\)\\.downloadstring\\(.*|.*.*new-object system\\.net\\.webclient\\)\\.downloadfile\\(.*|.*.*new-object net\\.webclient\\)\\.downloadstring\\(.*|.*.*new-object net\\.webclient\\)\\.downloadfile\\(.*|.*.* iex\\(.*|.*.*WScript\\.shell.*|.*.* -nop .*|.*.* -noprofile .*|.*.* -decode .*|.*.* -enc .*)))'
```



