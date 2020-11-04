| Title                    | Suspicious In-Memory Module Execution       |
|:-------------------------|:------------------|
| **Description**          | Detects the access to processes by other suspicious processes which have reflectively loaded libraries in their memory space. An example is SilentTrinity C2 behaviour. Generally speaking, when Sysmon EventID 10 cannot reference a stack call to a dll loaded from disk (the standard way), it will display "UNKNOWN" as the module name. Usually this means the stack call points to a module that was reflectively loaded in memory. Adding to this, it is not common to see such few calls in the stack (ntdll.dll --> kernelbase.dll --> unknown) which essentially means that most of the functions required by the process to execute certain routines are already present in memory, not requiring any calls to external libraries. The latter should also be considered suspicious. |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0004: Privilege Escalation](https://attack.mitre.org/tactics/TA0004)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1055: Process Injection](https://attack.mitre.org/techniques/T1055)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0014_10_windows_sysmon_ProcessAccess](../Data_Needed/DN_0014_10_windows_sysmon_ProcessAccess.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1055: Process Injection](../Triggers/T1055.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Low</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://azure.microsoft.com/en-ca/blog/detecting-in-memory-attacks-with-sysmon-and-azure-security-center/](https://azure.microsoft.com/en-ca/blog/detecting-in-memory-attacks-with-sysmon-and-azure-security-center/)</li></ul>  |
| **Author**               | Perez Diego (@darkquassar), oscd.community |


## Detection Rules

### Sigma rule

```
title: Suspicious In-Memory Module Execution
id: 5f113a8f-8b61-41ca-b90f-d374fa7e4a39
description: Detects the access to processes by other suspicious processes which have reflectively loaded libraries in their memory space. An example is SilentTrinity
    C2 behaviour. Generally speaking, when Sysmon EventID 10 cannot reference a stack call to a dll loaded from disk (the standard way), it will display "UNKNOWN"
    as the module name. Usually this means the stack call points to a module that was reflectively loaded in memory. Adding to this, it is not common to see such
    few calls in the stack (ntdll.dll --> kernelbase.dll --> unknown) which essentially means that most of the functions required by the process to execute certain
    routines are already present in memory, not requiring any calls to external libraries. The latter should also be considered suspicious.
status: experimental
date: 2019/10/27
author: Perez Diego (@darkquassar), oscd.community
references:
    - https://azure.microsoft.com/en-ca/blog/detecting-in-memory-attacks-with-sysmon-and-azure-security-center/
tags:
    - attack.privilege_escalation
    - attack.t1055
logsource:
    product: windows
    service: sysmon
detection:
    selection_01: 
        EventID: 10
        CallTrace: 
            - "C:\\Windows\\SYSTEM32\\ntdll.dll+*|C:\\Windows\\System32\\KERNELBASE.dll+*|UNKNOWN(*)"
            - "*UNKNOWN(*)|UNKNOWN(*)"
    selection_02: 
        EventID: 10
        CallTrace: "*UNKNOWN*"
    granted_access:
        GrantedAccess:
            - "0x1F0FFF"
            - "0x1F1FFF"
            - "0x143A"
            - "0x1410"
            - "0x1010"
            - "0x1F2FFF"
            - "0x1F3FFF"
            - "0x1FFFFF"
    condition: selection_01 OR (selection_02 AND granted_access)
fields:
    - ComputerName
    - User
    - SourceImage
    - TargetImage
    - CallTrace
level: critical
falsepositives:
    - Low

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "10" -and (($_.message -match "CallTrace.*C:\\Windows\\SYSTEM32\\ntdll.dll\+.*|C:\\Windows\\System32\\KERNELBASE.dll\+.*|UNKNOWN(.*)" -or $_.message -match "CallTrace.*.*UNKNOWN(.*)|UNKNOWN(.*)") -or ($_.message -match "CallTrace.*.*UNKNOWN.*" -and ($_.message -match "0x1F0FFF" -or $_.message -match "0x1F1FFF" -or $_.message -match "0x143A" -or $_.message -match "0x1410" -or $_.message -match "0x1010" -or $_.message -match "0x1F2FFF" -or $_.message -match "0x1F3FFF" -or $_.message -match "0x1FFFFF")))) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
(winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND winlog.event_id:"10" AND (winlog.event_data.CallTrace.keyword:(C\:\\Windows\\SYSTEM32\\ntdll.dll\+*|C\:\\Windows\\System32\\KERNELBASE.dll\+*|UNKNOWN\(*\) OR *UNKNOWN\(*\)|UNKNOWN\(*\)) OR (winlog.channel:"Microsoft\-Windows\-Sysmon\/Operational" AND winlog.event_data.CallTrace.keyword:*UNKNOWN* AND winlog.event_data.GrantedAccess:("0x1F0FFF" OR "0x1F1FFF" OR "0x143A" OR "0x1410" OR "0x1010" OR "0x1F2FFF" OR "0x1F3FFF" OR "0x1FFFFF"))))
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/5f113a8f-8b61-41ca-b90f-d374fa7e4a39 <<EOF
{
  "metadata": {
    "title": "Suspicious In-Memory Module Execution",
    "description": "Detects the access to processes by other suspicious processes which have reflectively loaded libraries in their memory space. An example is SilentTrinity C2 behaviour. Generally speaking, when Sysmon EventID 10 cannot reference a stack call to a dll loaded from disk (the standard way), it will display \"UNKNOWN\" as the module name. Usually this means the stack call points to a module that was reflectively loaded in memory. Adding to this, it is not common to see such few calls in the stack (ntdll.dll --> kernelbase.dll --> unknown) which essentially means that most of the functions required by the process to execute certain routines are already present in memory, not requiring any calls to external libraries. The latter should also be considered suspicious.",
    "tags": [
      "attack.privilege_escalation",
      "attack.t1055"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"10\" AND (winlog.event_data.CallTrace.keyword:(C\\:\\\\Windows\\\\SYSTEM32\\\\ntdll.dll\\+*|C\\:\\\\Windows\\\\System32\\\\KERNELBASE.dll\\+*|UNKNOWN\\(*\\) OR *UNKNOWN\\(*\\)|UNKNOWN\\(*\\)) OR (winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_data.CallTrace.keyword:*UNKNOWN* AND winlog.event_data.GrantedAccess:(\"0x1F0FFF\" OR \"0x1F1FFF\" OR \"0x143A\" OR \"0x1410\" OR \"0x1010\" OR \"0x1F2FFF\" OR \"0x1F3FFF\" OR \"0x1FFFFF\"))))"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"10\" AND (winlog.event_data.CallTrace.keyword:(C\\:\\\\Windows\\\\SYSTEM32\\\\ntdll.dll\\+*|C\\:\\\\Windows\\\\System32\\\\KERNELBASE.dll\\+*|UNKNOWN\\(*\\) OR *UNKNOWN\\(*\\)|UNKNOWN\\(*\\)) OR (winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_data.CallTrace.keyword:*UNKNOWN* AND winlog.event_data.GrantedAccess:(\"0x1F0FFF\" OR \"0x1F1FFF\" OR \"0x143A\" OR \"0x1410\" OR \"0x1010\" OR \"0x1F2FFF\" OR \"0x1F3FFF\" OR \"0x1FFFFF\"))))",
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
        "subject": "Sigma Rule 'Suspicious In-Memory Module Execution'",
        "body": "Hits:\n{{#ctx.payload.hits.hits}}Hit on {{_source.@timestamp}}:\nComputerName = {{_source.ComputerName}}\n        User = {{_source.User}}\n SourceImage = {{_source.SourceImage}}\n TargetImage = {{_source.TargetImage}}\n   CallTrace = {{_source.CallTrace}}================================================================================\n{{/ctx.payload.hits.hits}}",
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
(EventID:"10" AND (CallTrace.keyword:(C\:\\Windows\\SYSTEM32\\ntdll.dll\+*|C\:\\Windows\\System32\\KERNELBASE.dll\+*|UNKNOWN\(*\) *UNKNOWN\(*\)|UNKNOWN\(*\)) OR (CallTrace.keyword:*UNKNOWN* AND GrantedAccess:("0x1F0FFF" "0x1F1FFF" "0x143A" "0x1410" "0x1010" "0x1F2FFF" "0x1F3FFF" "0x1FFFFF"))))
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="10" ((CallTrace="C:\\Windows\\SYSTEM32\\ntdll.dll+*|C:\\Windows\\System32\\KERNELBASE.dll+*|UNKNOWN(*)" OR CallTrace="*UNKNOWN(*)|UNKNOWN(*)") OR (source="WinEventLog:Microsoft-Windows-Sysmon/Operational" CallTrace="*UNKNOWN*" (GrantedAccess="0x1F0FFF" OR GrantedAccess="0x1F1FFF" OR GrantedAccess="0x143A" OR GrantedAccess="0x1410" OR GrantedAccess="0x1010" OR GrantedAccess="0x1F2FFF" OR GrantedAccess="0x1F3FFF" OR GrantedAccess="0x1FFFFF")))) | table ComputerName,User,SourceImage,TargetImage,CallTrace
```


### logpoint
    
```
(event_id="10" (CallTrace IN ["C:\\Windows\\SYSTEM32\\ntdll.dll+*|C:\\Windows\\System32\\KERNELBASE.dll+*|UNKNOWN(*)", "*UNKNOWN(*)|UNKNOWN(*)"] OR (CallTrace="*UNKNOWN*" GrantedAccess IN ["0x1F0FFF", "0x1F1FFF", "0x143A", "0x1410", "0x1010", "0x1F2FFF", "0x1F3FFF", "0x1FFFFF"])))
```


### grep
    
```
grep -P '^(?:.*(?=.*10)(?=.*(?:.*(?:.*(?:.*C:\Windows\SYSTEM32\ntdll\.dll\+.*\|C:\Windows\System32\KERNELBASE\.dll\+.*\|UNKNOWN\(.*\)|.*.*UNKNOWN\(.*\)\|UNKNOWN\(.*\))|.*(?:.*(?=.*.*UNKNOWN.*)(?=.*(?:.*0x1F0FFF|.*0x1F1FFF|.*0x143A|.*0x1410|.*0x1010|.*0x1F2FFF|.*0x1F3FFF|.*0x1FFFFF)))))))'
```



