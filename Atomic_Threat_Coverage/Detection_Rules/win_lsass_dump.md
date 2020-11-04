| Title                    | LSASS Memory Dumping       |
|:-------------------------|:------------------|
| **Description**          | Detect creation of dump files containing the memory space of lsass.exe, which contains sensitive credentials. Identifies usage of Sysinternals procdump.exe to export the memory space of lsass.exe which contains sensitive credentials. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0006: Credential Access](https://attack.mitre.org/tactics/TA0006)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1003: OS Credential Dumping](https://attack.mitre.org/techniques/T1003)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1003: OS Credential Dumping](../Triggers/T1003.md)</li></ul>  |
| **Severity Level**       | high |
| **False Positives**      | <ul><li>Unlikely</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://eqllib.readthedocs.io/en/latest/analytics/1e1ef6be-12fc-11e9-8d76-4d6bb837cda4.html](https://eqllib.readthedocs.io/en/latest/analytics/1e1ef6be-12fc-11e9-8d76-4d6bb837cda4.html)</li><li>[https://eqllib.readthedocs.io/en/latest/analytics/210b4ea4-12fc-11e9-8d76-4d6bb837cda4.html](https://eqllib.readthedocs.io/en/latest/analytics/210b4ea4-12fc-11e9-8d76-4d6bb837cda4.html)</li><li>[https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1003/T1003.yaml](https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1003/T1003.yaml)</li></ul>  |
| **Author**               | E.M. Anhaus (orignally from Atomic Blue Detections, Tony Lambert), oscd.community |


## Detection Rules

### Sigma rule

```
title: LSASS Memory Dumping
id: ffa6861c-4461-4f59-8a41-578c39f3f23e
description: Detect creation of dump files containing the memory space of lsass.exe, which contains sensitive credentials. Identifies usage of Sysinternals procdump.exe
    to export the memory space of lsass.exe which contains sensitive credentials.
status: experimental
author: E.M. Anhaus (orignally from Atomic Blue Detections, Tony Lambert), oscd.community
date: 2019/10/24
modified: 2019/11/11
references:
    - https://eqllib.readthedocs.io/en/latest/analytics/1e1ef6be-12fc-11e9-8d76-4d6bb837cda4.html
    - https://eqllib.readthedocs.io/en/latest/analytics/210b4ea4-12fc-11e9-8d76-4d6bb837cda4.html
    - https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1003/T1003.yaml
tags:
    - attack.credential_access
    - attack.t1003
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        CommandLine|contains|all:
            - 'lsass'
            - '.dmp'
    selection2:
        Image|endswith: '\werfault.exe'
    selection3:
        Image|contains: '\procdump'
        Image|endswith: '.exe'
        CommandLine|contains: 'lsass'
    condition: selection1 and not selection2 or selection3
fields:
    - ComputerName
    - User
    - CommandLine
falsepositives:
    - Unlikely
level: high

```





### powershell
    
```
Get-WinEvent | where {((($_.message -match "CommandLine.*.*lsass.*" -and $_.message -match "CommandLine.*.*.dmp.*") -and  -not ($_.message -match "Image.*.*\\werfault.exe")) -or ($_.message -match "Image.*.*\\procdump.*" -and $_.message -match "Image.*.*.exe" -and $_.message -match "CommandLine.*.*lsass.*")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(((winlog.event_data.CommandLine.keyword:*lsass* AND winlog.event_data.CommandLine.keyword:*.dmp*) AND (NOT (winlog.event_data.Image.keyword:*\\werfault.exe))) OR (winlog.event_data.Image.keyword:*\\procdump* AND winlog.event_data.Image.keyword:*.exe AND winlog.event_data.CommandLine.keyword:*lsass*))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/ffa6861c-4461-4f59-8a41-578c39f3f23e <<EOF
{
  "metadata": {
    "title": "LSASS Memory Dumping",
    "description": "Detect creation of dump files containing the memory space of lsass.exe, which contains sensitive credentials. Identifies usage of Sysinternals procdump.exe to export the memory space of lsass.exe which contains sensitive credentials.",
    "tags": [
      "attack.credential_access",
      "attack.t1003"
    ],
    "query": "(((winlog.event_data.CommandLine.keyword:*lsass* AND winlog.event_data.CommandLine.keyword:*.dmp*) AND (NOT (winlog.event_data.Image.keyword:*\\\\werfault.exe))) OR (winlog.event_data.Image.keyword:*\\\\procdump* AND winlog.event_data.Image.keyword:*.exe AND winlog.event_data.CommandLine.keyword:*lsass*))"
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
                    "query": "(((winlog.event_data.CommandLine.keyword:*lsass* AND winlog.event_data.CommandLine.keyword:*.dmp*) AND (NOT (winlog.event_data.Image.keyword:*\\\\werfault.exe))) OR (winlog.event_data.Image.keyword:*\\\\procdump* AND winlog.event_data.Image.keyword:*.exe AND winlog.event_data.CommandLine.keyword:*lsass*))",
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
        "subject": "Sigma Rule 'LSASS Memory Dumping'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\nComputerName = {{_source.ComputerName}}\n        User = {{_source.User}}\n CommandLine = {{_source.CommandLine}}================================================================================\n{{/ctx.payload.hits.hits}}",
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
(((CommandLine.keyword:*lsass* AND CommandLine.keyword:*.dmp*) AND (NOT (Image.keyword:*\\werfault.exe))) OR (Image.keyword:*\\procdump* AND Image.keyword:*.exe AND CommandLine.keyword:*lsass*))
```


### splunk
    
```
(((CommandLine="*lsass*" CommandLine="*.dmp*") NOT (Image="*\\werfault.exe")) OR (Image="*\\procdump*" Image="*.exe" CommandLine="*lsass*")) | table ComputerName,User,CommandLine
```


### logpoint
    
```
(((CommandLine="*lsass*" CommandLine="*.dmp*")  -(Image="*\\werfault.exe")) OR (Image="*\\procdump*" Image="*.exe" CommandLine="*lsass*"))
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?=.*(?:.*(?=.*.*lsass.*)(?=.*.*\.dmp.*)))(?=.*(?!.*(?:.*(?=.*.*\werfault\.exe)))))|.*(?:.*(?=.*.*\procdump.*)(?=.*.*\.exe)(?=.*.*lsass.*))))'
```



