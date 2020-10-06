| Title                    | MSHTA Suspicious Execution 01       |
|:-------------------------|:------------------|
| **Description**          | Detection for mshta.exe suspicious execution patterns sometimes involving file polyglotism |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1140: Deobfuscate/Decode Files or Information](https://attack.mitre.org/techniques/T1140)</li><li>[T1218.005: Mshta](https://attack.mitre.org/techniques/T1218/005)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1140: Deobfuscate/Decode Files or Information](../Triggers/T1140.md)</li><li>[T1218.005: Mshta](../Triggers/T1218.005.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>False positives depend on scripts and administrative tools used in the monitored environment</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[http://blog.sevagas.com/?Hacking-around-HTA-files](http://blog.sevagas.com/?Hacking-around-HTA-files)</li><li>[https://0x00sec.org/t/clientside-exploitation-in-2018-how-pentesting-has-changed/7356](https://0x00sec.org/t/clientside-exploitation-in-2018-how-pentesting-has-changed/7356)</li><li>[https://docs.microsoft.com/en-us/dotnet/standard/data/xml/xslt-stylesheet-scripting-using-msxsl-script](https://docs.microsoft.com/en-us/dotnet/standard/data/xml/xslt-stylesheet-scripting-using-msxsl-script)</li><li>[https://medium.com/tsscyber/pentesting-and-hta-bypassing-powershell-constrained-language-mode-53a42856c997](https://medium.com/tsscyber/pentesting-and-hta-bypassing-powershell-constrained-language-mode-53a42856c997)</li></ul>  |
| **Author**               | Diego Perez (@darkquassar), Markus Neis, Swisscom (Improve Rule) |


## Detection Rules

### Sigma rule

```
title: MSHTA Suspicious Execution 01
id: cc7abbd0-762b-41e3-8a26-57ad50d2eea3
status: experimental
description: Detection for mshta.exe suspicious execution patterns sometimes involving file polyglotism
date: 2019/02/22
modified: 2020/08/23
author: Diego Perez (@darkquassar), Markus Neis, Swisscom (Improve Rule)
references:
    - http://blog.sevagas.com/?Hacking-around-HTA-files
    - https://0x00sec.org/t/clientside-exploitation-in-2018-how-pentesting-has-changed/7356
    - https://docs.microsoft.com/en-us/dotnet/standard/data/xml/xslt-stylesheet-scripting-using-msxsl-script
    - https://medium.com/tsscyber/pentesting-and-hta-bypassing-powershell-constrained-language-mode-53a42856c997
tags:
    - attack.defense_evasion
    - attack.t1140
    - attack.t1218.005
logsource:
    category: process_creation
    product: windows
falsepositives: 
    - False positives depend on scripts and administrative tools used in the monitored environment
level: high
detection:
    selection1:
        Image: '*\mshta.exe'
        CommandLine: 
            - '*vbscript*' 
            - '*.jpg*'
            - '*.png*'
            - '*.lnk*'
            # - '*.chm*'  # could be prone to false positives
            - '*.xls*'
            - '*.doc*'
            - '*.zip*'
    condition:
        selection1 

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "Image.*.*\\\\mshta.exe" -and ($_.message -match "CommandLine.*.*vbscript.*" -or $_.message -match "CommandLine.*.*.jpg.*" -or $_.message -match "CommandLine.*.*.png.*" -or $_.message -match "CommandLine.*.*.lnk.*" -or $_.message -match "CommandLine.*.*.xls.*" -or $_.message -match "CommandLine.*.*.doc.*" -or $_.message -match "CommandLine.*.*.zip.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.event_data.Image.keyword:*\\\\mshta.exe AND winlog.event_data.CommandLine.keyword:(*vbscript* OR *.jpg* OR *.png* OR *.lnk* OR *.xls* OR *.doc* OR *.zip*))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/cc7abbd0-762b-41e3-8a26-57ad50d2eea3 <<EOF\n{\n  "metadata": {\n    "title": "MSHTA Suspicious Execution 01",\n    "description": "Detection for mshta.exe suspicious execution patterns sometimes involving file polyglotism",\n    "tags": [\n      "attack.defense_evasion",\n      "attack.t1140",\n      "attack.t1218.005"\n    ],\n    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\mshta.exe AND winlog.event_data.CommandLine.keyword:(*vbscript* OR *.jpg* OR *.png* OR *.lnk* OR *.xls* OR *.doc* OR *.zip*))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "(winlog.event_data.Image.keyword:*\\\\\\\\mshta.exe AND winlog.event_data.CommandLine.keyword:(*vbscript* OR *.jpg* OR *.png* OR *.lnk* OR *.xls* OR *.doc* OR *.zip*))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'MSHTA Suspicious Execution 01\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
(Image.keyword:*\\\\mshta.exe AND CommandLine.keyword:(*vbscript* *.jpg* *.png* *.lnk* *.xls* *.doc* *.zip*))
```


### splunk
    
```
(Image="*\\\\mshta.exe" (CommandLine="*vbscript*" OR CommandLine="*.jpg*" OR CommandLine="*.png*" OR CommandLine="*.lnk*" OR CommandLine="*.xls*" OR CommandLine="*.doc*" OR CommandLine="*.zip*"))
```


### logpoint
    
```
(Image="*\\\\mshta.exe" CommandLine IN ["*vbscript*", "*.jpg*", "*.png*", "*.lnk*", "*.xls*", "*.doc*", "*.zip*"])
```


### grep
    
```
grep -P '^(?:.*(?=.*.*\\mshta\\.exe)(?=.*(?:.*.*vbscript.*|.*.*\\.jpg.*|.*.*\\.png.*|.*.*\\.lnk.*|.*.*\\.xls.*|.*.*\\.doc.*|.*.*\\.zip.*)))'
```



