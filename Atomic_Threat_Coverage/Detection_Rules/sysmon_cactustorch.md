| Title                    | CACTUSTORCH Remote Thread Creation       |
|:-------------------------|:------------------|
| **Description**          | Detects remote thread creation from CACTUSTORCH as described in references. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1055: Process Injection](https://attack.mitre.org/techniques/T1055)</li><li>[T1064: Scripting](https://attack.mitre.org/techniques/T1064)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0012_8_windows_sysmon_CreateRemoteThread](../Data_Needed/DN_0012_8_windows_sysmon_CreateRemoteThread.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1055: Process Injection](../Triggers/T1055.md)</li><li>[T1064: Scripting](../Triggers/T1064.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://twitter.com/SBousseaden/status/1090588499517079552](https://twitter.com/SBousseaden/status/1090588499517079552)</li><li>[https://github.com/mdsecactivebreach/CACTUSTORCH](https://github.com/mdsecactivebreach/CACTUSTORCH)</li></ul>  |
| **Author**               | @SBousseaden (detection), Thomas Patzke (rule) |


## Detection Rules

### Sigma rule

```
title: CACTUSTORCH Remote Thread Creation
id: 2e4e488a-6164-4811-9ea1-f960c7359c40
description: Detects remote thread creation from CACTUSTORCH as described in references.
references:
    - https://twitter.com/SBousseaden/status/1090588499517079552
    - https://github.com/mdsecactivebreach/CACTUSTORCH
status: experimental
author: '@SBousseaden (detection), Thomas Patzke (rule)'
date: 2019/02/01
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 8
        SourceImage:
            - '*\System32\cscript.exe'
            - '*\System32\wscript.exe'
            - '*\System32\mshta.exe'
            - '*\winword.exe'
            - '*\excel.exe'
        TargetImage: '*\SysWOW64\\*'
        StartModule: null
    condition: selection
tags:
    - attack.execution
    - attack.t1055
    - attack.t1064
falsepositives:
    - unknown
level: high

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "8" -and ($_.message -match "SourceImage.*.*\\System32\\cscript.exe" -or $_.message -match "SourceImage.*.*\\System32\\wscript.exe" -or $_.message -match "SourceImage.*.*\\System32\\mshta.exe" -or $_.message -match "SourceImage.*.*\\winword.exe" -or $_.message -match "SourceImage.*.*\\excel.exe") -and $_.message -match "TargetImage.*.*\\SysWOW64\\.*" -and -not StartModule="*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND winlog.event_id:"8" AND winlog.event_data.SourceImage.keyword:(*\\System32\\cscript.exe OR *\\System32\\wscript.exe OR *\\System32\\mshta.exe OR *\\winword.exe OR *\\excel.exe) AND winlog.event_data.TargetImage.keyword:*\\SysWOW64\\* AND NOT _exists_:winlog.event_data.StartModule)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/2e4e488a-6164-4811-9ea1-f960c7359c40 <<EOF
{
  "metadata": {
    "title": "CACTUSTORCH Remote Thread Creation",
    "description": "Detects remote thread creation from CACTUSTORCH as described in references.",
    "tags": [
      "attack.execution",
      "attack.t1055",
      "attack.t1064"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"8\" AND winlog.event_data.SourceImage.keyword:(*\\\\System32\\\\cscript.exe OR *\\\\System32\\\\wscript.exe OR *\\\\System32\\\\mshta.exe OR *\\\\winword.exe OR *\\\\excel.exe) AND winlog.event_data.TargetImage.keyword:*\\\\SysWOW64\\\\* AND NOT _exists_:winlog.event_data.StartModule)"
  },
  "trigger": {
    "schedule": {
      "interval": "30m"
    }
  },
  "input": {
    "search": {
      "request": {
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "must": [
                {
                  "query_string": {
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"8\" AND winlog.event_data.SourceImage.keyword:(*\\\\System32\\\\cscript.exe OR *\\\\System32\\\\wscript.exe OR *\\\\System32\\\\mshta.exe OR *\\\\winword.exe OR *\\\\excel.exe) AND winlog.event_data.TargetImage.keyword:*\\\\SysWOW64\\\\* AND NOT _exists_:winlog.event_data.StartModule)",
                    "analyze_wildcard": true
                  }
                }
              ],
              "filter": {
                "range": {
                  "timestamp": {
                    "gte": "now-30m/m"
                  }
                }
              }
            }
          }
        },
        "indices": [
          "winlogbeat-*"
        ]
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "not_eq": 0
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "to": "root@localhost",
        "subject": "Sigma Rule 'CACTUSTORCH Remote Thread Creation'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}{{_source}}\n================================================================================\n{{/ctx.payload.hits.hits}}",
        "attachments": {
          "data.json": {
            "data": {
              "format": "json"
            }
          }
        }
      }
    }
  }
}
EOF

```


### graylog
    
```
(EventID:"8" AND SourceImage.keyword:(*\\System32\\cscript.exe *\\System32\\wscript.exe *\\System32\\mshta.exe *\\winword.exe *\\excel.exe) AND TargetImage.keyword:*\\SysWOW64\\* AND NOT _exists_:StartModule)
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="8" (SourceImage="*\\System32\\cscript.exe" OR SourceImage="*\\System32\\wscript.exe" OR SourceImage="*\\System32\\mshta.exe" OR SourceImage="*\\winword.exe" OR SourceImage="*\\excel.exe") TargetImage="*\\SysWOW64\\*" NOT StartModule="*")
```


### logpoint
    
```
(event_id="8" SourceImage IN ["*\\System32\\cscript.exe", "*\\System32\\wscript.exe", "*\\System32\\mshta.exe", "*\\winword.exe", "*\\excel.exe"] TargetImage="*\\SysWOW64\\*" -StartModule=*)
```


### grep
    
```
grep -P '^(?:.*(?=.*8)(?=.*(?:.*.*\System32\cscript\.exe|.*.*\System32\wscript\.exe|.*.*\System32\mshta\.exe|.*.*\winword\.exe|.*.*\excel\.exe))(?=.*.*\SysWOW64\\.*)(?=.*(?!StartModule)))'
```



