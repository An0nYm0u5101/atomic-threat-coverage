| Title                    | Greenbug Campaign Indicators       |
|:-------------------------|:------------------|
| **Description**          | Detects tools and process executions as observed in a Greenbug campaign in May 2020 |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0002: Execution](https://attack.mitre.org/tactics/TA0002)</li><li>[TA0011: Command and Control](https://attack.mitre.org/tactics/TA0011)</li><li>[TA0005: Defense Evasion](https://attack.mitre.org/tactics/TA0005)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1059.001: PowerShell](https://attack.mitre.org/techniques/T1059/001)</li><li>[T1086: PowerShell](https://attack.mitre.org/techniques/T1086)</li><li>[T1105: Ingress Tool Transfer](https://attack.mitre.org/techniques/T1105)</li><li>[T1036: Masquerading](https://attack.mitre.org/techniques/T1036)</li><li>[T1036.005: Match Legitimate Name or Location](https://attack.mitre.org/techniques/T1036/005)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              | <ul><li>[T1059.001: PowerShell](../Triggers/T1059.001.md)</li><li>[T1105: Ingress Tool Transfer](../Triggers/T1105.md)</li></ul>  |
| **Severity Level**       | critical |
| **False Positives**      | <ul><li>Unknown</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://symantec-enterprise-blogs.security.com/blogs/threat-intelligence/greenbug-espionage-telco-south-asia](https://symantec-enterprise-blogs.security.com/blogs/threat-intelligence/greenbug-espionage-telco-south-asia)</li></ul>  |
| **Author**               | Florian Roth |
| Other Tags           | <ul><li>attack.g0049</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Greenbug Campaign Indicators
id: 3711eee4-a808-4849-8a14-faf733da3612
status: experimental
description: Detects tools and process executions as observed in a Greenbug campaign in May 2020
references:
    - https://symantec-enterprise-blogs.security.com/blogs/threat-intelligence/greenbug-espionage-telco-south-asia
author: Florian Roth
date: 2020/05/20
modified: 2020/08/27
tags:
    - attack.g0049
    - attack.execution
    - attack.t1059.001
    - attack.t1086  #an old one
    - attack.command_and_control
    - attack.t1105
    - attack.defense_evasion
    - attack.t1036  # an old one
    - attack.t1036.005
logsource:
    category: process_creation
    product: windows
detection:
    selection1:
        CommandLine|contains|all: 
            - 'bitsadmin /transfer'
            - 'CSIDL_APPDATA'
    selection2:
        CommandLine|contains: 
            - 'CSIDL_SYSTEM_DRIVE'
    selection3: 
        CommandLine|contains:
            - '\msf.ps1'
            - '8989 -e cmd.exe'
            - 'system.Data.SqlClient.SqlDataAdapter($cmd); [void]$da.fill'
            - '-nop -w hidden -c $k=new-object'
            - '[Net.CredentialCache]::DefaultCredentials;IEX '
            - ' -nop -w hidden -c $m=new-object net.webclient;$m'
            - '-noninteractive -executionpolicy bypass whoami'
            - '-noninteractive -executionpolicy bypass netstat -a'
            - 'L3NlcnZlc'  # base64 encoded '/server='
    selection4:
        Image|endswith:
            - '\adobe\Adobe.exe'
            - '\oracle\local.exe'
            - '\revshell.exe'
            - 'infopagesbackup\ncat.exe'
            - 'CSIDL_SYSTEM\cmd.exe'
            - '\programdata\oracle\java.exe'
            - 'CSIDL_COMMON_APPDATA\comms\comms.exe'
            - '\Programdata\VMware\Vmware.exe'
    condition: 1 of them
falsepositives:
    - Unknown
level: critical

```





### powershell
    
```
Get-WinEvent | where {(($_.message -match "CommandLine.*.*bitsadmin /transfer.*" -and $_.message -match "CommandLine.*.*CSIDL_APPDATA.*") -or ($_.message -match "CommandLine.*.*CSIDL_SYSTEM_DRIVE.*") -or ($_.message -match "CommandLine.*.*\\\\msf.ps1.*" -or $_.message -match "CommandLine.*.*8989 -e cmd.exe.*" -or $_.message -match "CommandLine.*.*system.Data.SqlClient.SqlDataAdapter($cmd); [void]$da.fill.*" -or $_.message -match "CommandLine.*.*-nop -w hidden -c $k=new-object.*" -or $_.message -match "CommandLine.*.*[Net.CredentialCache]::DefaultCredentials;IEX .*" -or $_.message -match "CommandLine.*.* -nop -w hidden -c $m=new-object net.webclient;$m.*" -or $_.message -match "CommandLine.*.*-noninteractive -executionpolicy bypass whoami.*" -or $_.message -match "CommandLine.*.*-noninteractive -executionpolicy bypass netstat -a.*" -or $_.message -match "CommandLine.*.*L3NlcnZlc.*") -or ($_.message -match "Image.*.*\\\\adobe\\\\Adobe.exe" -or $_.message -match "Image.*.*\\\\oracle\\\\local.exe" -or $_.message -match "Image.*.*\\\\revshell.exe" -or $_.message -match "Image.*.*infopagesbackup\\\\ncat.exe" -or $_.message -match "Image.*.*CSIDL_SYSTEM\\\\cmd.exe" -or $_.message -match "Image.*.*\\\\programdata\\\\oracle\\\\java.exe" -or $_.message -match "Image.*.*CSIDL_COMMON_APPDATA\\\\comms\\\\comms.exe" -or $_.message -match "Image.*.*\\\\Programdata\\\\VMware\\\\Vmware.exe")) } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
((winlog.event_data.CommandLine.keyword:*bitsadmin\\ \\/transfer* AND winlog.event_data.CommandLine.keyword:*CSIDL_APPDATA*) OR winlog.event_data.CommandLine.keyword:(*CSIDL_SYSTEM_DRIVE*) OR winlog.event_data.CommandLine.keyword:(*\\\\msf.ps1* OR *8989\\ \\-e\\ cmd.exe* OR *system.Data.SqlClient.SqlDataAdapter\\($cmd\\);\\ \\[void\\]$da.fill* OR *\\-nop\\ \\-w\\ hidden\\ \\-c\\ $k\\=new\\-object* OR *\\[Net.CredentialCache\\]\\:\\:DefaultCredentials;IEX\\ * OR *\\ \\-nop\\ \\-w\\ hidden\\ \\-c\\ $m\\=new\\-object\\ net.webclient;$m* OR *\\-noninteractive\\ \\-executionpolicy\\ bypass\\ whoami* OR *\\-noninteractive\\ \\-executionpolicy\\ bypass\\ netstat\\ \\-a* OR *L3NlcnZlc*) OR winlog.event_data.Image.keyword:(*\\\\adobe\\\\Adobe.exe OR *\\\\oracle\\\\local.exe OR *\\\\revshell.exe OR *infopagesbackup\\\\ncat.exe OR *CSIDL_SYSTEM\\\\cmd.exe OR *\\\\programdata\\\\oracle\\\\java.exe OR *CSIDL_COMMON_APPDATA\\\\comms\\\\comms.exe OR *\\\\Programdata\\\\VMware\\\\Vmware.exe))
```


### xpack-watcher
    
```
curl -s -XPUT -H \'Content-Type: application/json\' --data-binary @- localhost:9200/_watcher/watch/3711eee4-a808-4849-8a14-faf733da3612 <<EOF\n{\n  "metadata": {\n    "title": "Greenbug Campaign Indicators",\n    "description": "Detects tools and process executions as observed in a Greenbug campaign in May 2020",\n    "tags": [\n      "attack.g0049",\n      "attack.execution",\n      "attack.t1059.001",\n      "attack.t1086",\n      "attack.command_and_control",\n      "attack.t1105",\n      "attack.defense_evasion",\n      "attack.t1036",\n      "attack.t1036.005"\n    ],\n    "query": "((winlog.event_data.CommandLine.keyword:*bitsadmin\\\\ \\\\/transfer* AND winlog.event_data.CommandLine.keyword:*CSIDL_APPDATA*) OR winlog.event_data.CommandLine.keyword:(*CSIDL_SYSTEM_DRIVE*) OR winlog.event_data.CommandLine.keyword:(*\\\\\\\\msf.ps1* OR *8989\\\\ \\\\-e\\\\ cmd.exe* OR *system.Data.SqlClient.SqlDataAdapter\\\\($cmd\\\\);\\\\ \\\\[void\\\\]$da.fill* OR *\\\\-nop\\\\ \\\\-w\\\\ hidden\\\\ \\\\-c\\\\ $k\\\\=new\\\\-object* OR *\\\\[Net.CredentialCache\\\\]\\\\:\\\\:DefaultCredentials;IEX\\\\ * OR *\\\\ \\\\-nop\\\\ \\\\-w\\\\ hidden\\\\ \\\\-c\\\\ $m\\\\=new\\\\-object\\\\ net.webclient;$m* OR *\\\\-noninteractive\\\\ \\\\-executionpolicy\\\\ bypass\\\\ whoami* OR *\\\\-noninteractive\\\\ \\\\-executionpolicy\\\\ bypass\\\\ netstat\\\\ \\\\-a* OR *L3NlcnZlc*) OR winlog.event_data.Image.keyword:(*\\\\\\\\adobe\\\\\\\\Adobe.exe OR *\\\\\\\\oracle\\\\\\\\local.exe OR *\\\\\\\\revshell.exe OR *infopagesbackup\\\\\\\\ncat.exe OR *CSIDL_SYSTEM\\\\\\\\cmd.exe OR *\\\\\\\\programdata\\\\\\\\oracle\\\\\\\\java.exe OR *CSIDL_COMMON_APPDATA\\\\\\\\comms\\\\\\\\comms.exe OR *\\\\\\\\Programdata\\\\\\\\VMware\\\\\\\\Vmware.exe))"\n  },\n  "trigger": {\n    "schedule": {\n      "interval": "30m"\n    }\n  },\n  "input": {\n    "search": {\n      "request": {\n        "body": {\n          "size": 0,\n          "query": {\n            "bool": {\n              "must": [\n                {\n                  "query_string": {\n                    "query": "((winlog.event_data.CommandLine.keyword:*bitsadmin\\\\ \\\\/transfer* AND winlog.event_data.CommandLine.keyword:*CSIDL_APPDATA*) OR winlog.event_data.CommandLine.keyword:(*CSIDL_SYSTEM_DRIVE*) OR winlog.event_data.CommandLine.keyword:(*\\\\\\\\msf.ps1* OR *8989\\\\ \\\\-e\\\\ cmd.exe* OR *system.Data.SqlClient.SqlDataAdapter\\\\($cmd\\\\);\\\\ \\\\[void\\\\]$da.fill* OR *\\\\-nop\\\\ \\\\-w\\\\ hidden\\\\ \\\\-c\\\\ $k\\\\=new\\\\-object* OR *\\\\[Net.CredentialCache\\\\]\\\\:\\\\:DefaultCredentials;IEX\\\\ * OR *\\\\ \\\\-nop\\\\ \\\\-w\\\\ hidden\\\\ \\\\-c\\\\ $m\\\\=new\\\\-object\\\\ net.webclient;$m* OR *\\\\-noninteractive\\\\ \\\\-executionpolicy\\\\ bypass\\\\ whoami* OR *\\\\-noninteractive\\\\ \\\\-executionpolicy\\\\ bypass\\\\ netstat\\\\ \\\\-a* OR *L3NlcnZlc*) OR winlog.event_data.Image.keyword:(*\\\\\\\\adobe\\\\\\\\Adobe.exe OR *\\\\\\\\oracle\\\\\\\\local.exe OR *\\\\\\\\revshell.exe OR *infopagesbackup\\\\\\\\ncat.exe OR *CSIDL_SYSTEM\\\\\\\\cmd.exe OR *\\\\\\\\programdata\\\\\\\\oracle\\\\\\\\java.exe OR *CSIDL_COMMON_APPDATA\\\\\\\\comms\\\\\\\\comms.exe OR *\\\\\\\\Programdata\\\\\\\\VMware\\\\\\\\Vmware.exe))",\n                    "analyze_wildcard": true\n                  }\n                }\n              ],\n              "filter": {\n                "range": {\n                  "timestamp": {\n                    "gte": "now-30m/m"\n                  }\n                }\n              }\n            }\n          }\n        },\n        "indices": [\n          "winlogbeat-*"\n        ]\n      }\n    }\n  },\n  "condition": {\n    "compare": {\n      "ctx.payload.hits.total": {\n        "not_eq": 0\n      }\n    }\n  },\n  "actions": {\n    "send_email": {\n      "throttle_period": "15m",\n      "email": {\n        "profile": "standard",\n        "from": "root@localhost",\n        "to": "root@localhost",\n        "subject": "Sigma Rule \'Greenbug Campaign Indicators\'",\n        "body": "Hits:\\n{{#ctx.payload.hits.hits}}{{_source}}\\n================================================================================\\n{{/ctx.payload.hits.hits}}",\n        "attachments": {\n          "data.json": {\n            "data": {\n              "format": "json"\n            }\n          }\n        }\n      }\n    }\n  }\n}\nEOF\n
```


### graylog
    
```
((CommandLine.keyword:*bitsadmin \\/transfer* AND CommandLine.keyword:*CSIDL_APPDATA*) OR CommandLine.keyword:(*CSIDL_SYSTEM_DRIVE*) OR CommandLine.keyword:(*\\\\msf.ps1* *8989 \\-e cmd.exe* *system.Data.SqlClient.SqlDataAdapter\\($cmd\\); \\[void\\]$da.fill* *\\-nop \\-w hidden \\-c $k=new\\-object* *\\[Net.CredentialCache\\]\\:\\:DefaultCredentials;IEX * * \\-nop \\-w hidden \\-c $m=new\\-object net.webclient;$m* *\\-noninteractive \\-executionpolicy bypass whoami* *\\-noninteractive \\-executionpolicy bypass netstat \\-a* *L3NlcnZlc*) OR Image.keyword:(*\\\\adobe\\\\Adobe.exe *\\\\oracle\\\\local.exe *\\\\revshell.exe *infopagesbackup\\\\ncat.exe *CSIDL_SYSTEM\\\\cmd.exe *\\\\programdata\\\\oracle\\\\java.exe *CSIDL_COMMON_APPDATA\\\\comms\\\\comms.exe *\\\\Programdata\\\\VMware\\\\Vmware.exe))
```


### splunk
    
```
((CommandLine="*bitsadmin /transfer*" CommandLine="*CSIDL_APPDATA*") OR (CommandLine="*CSIDL_SYSTEM_DRIVE*") OR (CommandLine="*\\\\msf.ps1*" OR CommandLine="*8989 -e cmd.exe*" OR CommandLine="*system.Data.SqlClient.SqlDataAdapter($cmd); [void]$da.fill*" OR CommandLine="*-nop -w hidden -c $k=new-object*" OR CommandLine="*[Net.CredentialCache]::DefaultCredentials;IEX *" OR CommandLine="* -nop -w hidden -c $m=new-object net.webclient;$m*" OR CommandLine="*-noninteractive -executionpolicy bypass whoami*" OR CommandLine="*-noninteractive -executionpolicy bypass netstat -a*" OR CommandLine="*L3NlcnZlc*") OR (Image="*\\\\adobe\\\\Adobe.exe" OR Image="*\\\\oracle\\\\local.exe" OR Image="*\\\\revshell.exe" OR Image="*infopagesbackup\\\\ncat.exe" OR Image="*CSIDL_SYSTEM\\\\cmd.exe" OR Image="*\\\\programdata\\\\oracle\\\\java.exe" OR Image="*CSIDL_COMMON_APPDATA\\\\comms\\\\comms.exe" OR Image="*\\\\Programdata\\\\VMware\\\\Vmware.exe"))
```


### logpoint
    
```
((CommandLine="*bitsadmin /transfer*" CommandLine="*CSIDL_APPDATA*") OR CommandLine IN ["*CSIDL_SYSTEM_DRIVE*"] OR CommandLine IN ["*\\\\msf.ps1*", "*8989 -e cmd.exe*", "*system.Data.SqlClient.SqlDataAdapter($cmd); [void]$da.fill*", "*-nop -w hidden -c $k=new-object*", "*[Net.CredentialCache]::DefaultCredentials;IEX *", "* -nop -w hidden -c $m=new-object net.webclient;$m*", "*-noninteractive -executionpolicy bypass whoami*", "*-noninteractive -executionpolicy bypass netstat -a*", "*L3NlcnZlc*"] OR Image IN ["*\\\\adobe\\\\Adobe.exe", "*\\\\oracle\\\\local.exe", "*\\\\revshell.exe", "*infopagesbackup\\\\ncat.exe", "*CSIDL_SYSTEM\\\\cmd.exe", "*\\\\programdata\\\\oracle\\\\java.exe", "*CSIDL_COMMON_APPDATA\\\\comms\\\\comms.exe", "*\\\\Programdata\\\\VMware\\\\Vmware.exe"])
```


### grep
    
```
grep -P '^(?:.*(?:.*(?:.*(?=.*.*bitsadmin /transfer.*)(?=.*.*CSIDL_APPDATA.*))|.*(?:.*.*CSIDL_SYSTEM_DRIVE.*)|.*(?:.*.*\\msf\\.ps1.*|.*.*8989 -e cmd\\.exe.*|.*.*system\\.Data\\.SqlClient\\.SqlDataAdapter\\(\\$cmd\\); \\[void\\]\\$da\\.fill.*|.*.*-nop -w hidden -c \\$k=new-object.*|.*.*\\[Net\\.CredentialCache\\]::DefaultCredentials;IEX .*|.*.* -nop -w hidden -c \\$m=new-object net\\.webclient;\\$m.*|.*.*-noninteractive -executionpolicy bypass whoami.*|.*.*-noninteractive -executionpolicy bypass netstat -a.*|.*.*L3NlcnZlc.*)|.*(?:.*.*\\adobe\\Adobe\\.exe|.*.*\\oracle\\local\\.exe|.*.*\\revshell\\.exe|.*.*infopagesbackup\\ncat\\.exe|.*.*CSIDL_SYSTEM\\cmd\\.exe|.*.*\\programdata\\oracle\\java\\.exe|.*.*CSIDL_COMMON_APPDATA\\comms\\comms\\.exe|.*.*\\Programdata\\VMware\\Vmware\\.exe)))'
```



