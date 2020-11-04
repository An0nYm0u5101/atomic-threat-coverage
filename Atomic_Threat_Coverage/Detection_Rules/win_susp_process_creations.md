| Title                    | Suspicious Process Creation       |
|:-------------------------|:------------------|
| **Description**          | Detects suspicious process starts on Windows systems based on keywords |
| **ATT&amp;CK Tactic**    |   This Detection Rule wasn't mapped to ATT&amp;CK Tactic yet  |
| **ATT&amp;CK Technique** |  This Detection Rule wasn't mapped to ATT&amp;CK Technique yet  |
| **Data Needed**          | <ul><li>[DN_0002_4688_windows_process_creation_with_commandline](../Data_Needed/DN_0002_4688_windows_process_creation_with_commandline.md)</li><li>[DN_0003_1_windows_sysmon_process_creation](../Data_Needed/DN_0003_1_windows_sysmon_process_creation.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | medium |
| **False Positives**      | <ul><li>False positives depend on scripts and administrative tools used in the monitored environment</li></ul>  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://www.swordshield.com/2015/07/getting-hashes-from-ntds-dit-file/](https://www.swordshield.com/2015/07/getting-hashes-from-ntds-dit-file/)</li><li>[https://www.youtube.com/watch?v=H3t_kHQG1Js&feature=youtu.be&t=15m35s](https://www.youtube.com/watch?v=H3t_kHQG1Js&feature=youtu.be&t=15m35s)</li><li>[https://winscripting.blog/2017/05/12/first-entry-welcome-and-uac-bypass/](https://winscripting.blog/2017/05/12/first-entry-welcome-and-uac-bypass/)</li><li>[https://twitter.com/subTee/status/872244674609676288](https://twitter.com/subTee/status/872244674609676288)</li><li>[https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/remote-tool-examples](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/remote-tool-examples)</li><li>[https://tyranidslair.blogspot.ca/2017/07/dg-on-windows-10-s-executing-arbitrary.html](https://tyranidslair.blogspot.ca/2017/07/dg-on-windows-10-s-executing-arbitrary.html)</li><li>[https://www.trustedsec.com/2017/07/new-tool-release-nps_payload/](https://www.trustedsec.com/2017/07/new-tool-release-nps_payload/)</li><li>[https://subt0x10.blogspot.ca/2017/04/bypassing-application-whitelisting.html](https://subt0x10.blogspot.ca/2017/04/bypassing-application-whitelisting.html)</li><li>[https://gist.github.com/subTee/7937a8ef07409715f15b84781e180c46#file-rat-bat](https://gist.github.com/subTee/7937a8ef07409715f15b84781e180c46#file-rat-bat)</li><li>[https://twitter.com/vector_sec/status/896049052642533376](https://twitter.com/vector_sec/status/896049052642533376)</li><li>[http://security-research.dyndns.org/pub/slides/FIRST-TC-2018/FIRST-TC-2018_Tom-Ueltschi_Sysmon_PUBLIC.pdf](http://security-research.dyndns.org/pub/slides/FIRST-TC-2018/FIRST-TC-2018_Tom-Ueltschi_Sysmon_PUBLIC.pdf)</li></ul>  |
| **Author**               | Florian Roth, Daniil Yugoslavskiy, oscd.community (update) |
| Other Tags           | <ul><li>car.2013-07-001</li></ul> | 

## Detection Rules

### Sigma rule

```
title: Suspicious Process Creation
id: 5f0f47a5-cb16-4dbe-9e31-e8d976d73de3
description: Detects suspicious process starts on Windows systems based on keywords
status: experimental
references:
    - https://www.swordshield.com/2015/07/getting-hashes-from-ntds-dit-file/
    - https://www.youtube.com/watch?v=H3t_kHQG1Js&feature=youtu.be&t=15m35s
    - https://winscripting.blog/2017/05/12/first-entry-welcome-and-uac-bypass/
    - https://twitter.com/subTee/status/872244674609676288
    - https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/remote-tool-examples
    - https://tyranidslair.blogspot.ca/2017/07/dg-on-windows-10-s-executing-arbitrary.html
    - https://www.trustedsec.com/2017/07/new-tool-release-nps_payload/
    - https://subt0x10.blogspot.ca/2017/04/bypassing-application-whitelisting.html
    - https://gist.github.com/subTee/7937a8ef07409715f15b84781e180c46#file-rat-bat
    - https://twitter.com/vector_sec/status/896049052642533376
    - http://security-research.dyndns.org/pub/slides/FIRST-TC-2018/FIRST-TC-2018_Tom-Ueltschi_Sysmon_PUBLIC.pdf
author: Florian Roth, Daniil Yugoslavskiy, oscd.community (update)
date: 2018/01/01
modified: 2019/11/01
tags:
    - car.2013-07-001
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        CommandLine:
            - '* sekurlsa:*'
            - net localgroup administrators * /add
            - net group "Domain Admins" * /ADD /DOMAIN
            - certutil.exe *-urlcache* http*
            - certutil.exe *-urlcache* ftp*
            - netsh advfirewall firewall *\AppData\\*
            - attrib +S +H +R *\AppData\\*
            - schtasks* /create *\AppData\\*
            - schtasks* /sc minute*
            - '*\Regasm.exe *\AppData\\*'
            - '*\Regasm *\AppData\\*'
            - '*\bitsadmin* /transfer*'
            - '*\certutil.exe * -decode *'
            - '*\certutil.exe * -decodehex *'
            - '*\certutil.exe -ping *'
            - icacls * /grant Everyone:F /T /C /Q
            - '* wbadmin.exe delete catalog -quiet*'
            - '*\wscript.exe *.jse'
            - '*\wscript.exe *.js'
            - '*\wscript.exe *.vba'
            - '*\wscript.exe *.vbe'
            - '*\cscript.exe *.jse'
            - '*\cscript.exe *.js'
            - '*\cscript.exe *.vba'
            - '*\cscript.exe *.vbe'
            - '*\fodhelper.exe'
            - '*waitfor*/s*'
            - '*waitfor*/si persist*'
            - '*remote*/s*'
            - '*remote*/c*'
            - '*remote*/q*'
            - '*AddInProcess*'
            - '* /stext *'
            - '* /scomma *'
            - '* /stab *'
            - '* /stabular *'
            - '* /shtml *'
            - '* /sverhtml *'
            - '* /sxml *'
    condition: selection
fields:
    - ComputerName
    - User
    - CommandLine
falsepositives:
    - False positives depend on scripts and administrative tools used in the monitored environment
level: medium

```





### powershell
    
```
Get-WinEvent | where {($_.message -match "CommandLine.*.* sekurlsa:.*" -or $_.message -match "CommandLine.*net localgroup administrators .* /add" -or $_.message -match "CommandLine.*net group \"Domain Admins\" .* /ADD /DOMAIN" -or $_.message -match "CommandLine.*certutil.exe .*-urlcache.* http.*" -or $_.message -match "CommandLine.*certutil.exe .*-urlcache.* ftp.*" -or $_.message -match "CommandLine.*netsh advfirewall firewall .*\\AppData\\.*" -or $_.message -match "CommandLine.*attrib \+S \+H \+R .*\\AppData\\.*" -or $_.message -match "CommandLine.*schtasks.* /create .*\\AppData\\.*" -or $_.message -match "CommandLine.*schtasks.* /sc minute.*" -or $_.message -match "CommandLine.*.*\\Regasm.exe .*\\AppData\\.*" -or $_.message -match "CommandLine.*.*\\Regasm .*\\AppData\\.*" -or $_.message -match "CommandLine.*.*\\bitsadmin.* /transfer.*" -or $_.message -match "CommandLine.*.*\\certutil.exe .* -decode .*" -or $_.message -match "CommandLine.*.*\\certutil.exe .* -decodehex .*" -or $_.message -match "CommandLine.*.*\\certutil.exe -ping .*" -or $_.message -match "CommandLine.*icacls .* /grant Everyone:F /T /C /Q" -or $_.message -match "CommandLine.*.* wbadmin.exe delete catalog -quiet.*" -or $_.message -match "CommandLine.*.*\\wscript.exe .*.jse" -or $_.message -match "CommandLine.*.*\\wscript.exe .*.js" -or $_.message -match "CommandLine.*.*\\wscript.exe .*.vba" -or $_.message -match "CommandLine.*.*\\wscript.exe .*.vbe" -or $_.message -match "CommandLine.*.*\\cscript.exe .*.jse" -or $_.message -match "CommandLine.*.*\\cscript.exe .*.js" -or $_.message -match "CommandLine.*.*\\cscript.exe .*.vba" -or $_.message -match "CommandLine.*.*\\cscript.exe .*.vbe" -or $_.message -match "CommandLine.*.*\\fodhelper.exe" -or $_.message -match "CommandLine.*.*waitfor.*/s.*" -or $_.message -match "CommandLine.*.*waitfor.*/si persist.*" -or $_.message -match "CommandLine.*.*remote.*/s.*" -or $_.message -match "CommandLine.*.*remote.*/c.*" -or $_.message -match "CommandLine.*.*remote.*/q.*" -or $_.message -match "CommandLine.*.*AddInProcess.*" -or $_.message -match "CommandLine.*.* /stext .*" -or $_.message -match "CommandLine.*.* /scomma .*" -or $_.message -match "CommandLine.*.* /stab .*" -or $_.message -match "CommandLine.*.* /stabular .*" -or $_.message -match "CommandLine.*.* /shtml .*" -or $_.message -match "CommandLine.*.* /sverhtml .*" -or $_.message -match "CommandLine.*.* /sxml .*") } | select TimeCreated,Id,RecordId,ProcessId,MachineName,Message
```


### es-qs
    
```
winlog.event_data.CommandLine.keyword:(*\ sekurlsa\:* OR net\ localgroup\ administrators\ *\ \/add OR net\ group\ \"Domain\ Admins\"\ *\ \/ADD\ \/DOMAIN OR certutil.exe\ *\-urlcache*\ http* OR certutil.exe\ *\-urlcache*\ ftp* OR netsh\ advfirewall\ firewall\ *\\AppData\\* OR attrib\ \+S\ \+H\ \+R\ *\\AppData\\* OR schtasks*\ \/create\ *\\AppData\\* OR schtasks*\ \/sc\ minute* OR *\\Regasm.exe\ *\\AppData\\* OR *\\Regasm\ *\\AppData\\* OR *\\bitsadmin*\ \/transfer* OR *\\certutil.exe\ *\ \-decode\ * OR *\\certutil.exe\ *\ \-decodehex\ * OR *\\certutil.exe\ \-ping\ * OR icacls\ *\ \/grant\ Everyone\:F\ \/T\ \/C\ \/Q OR *\ wbadmin.exe\ delete\ catalog\ \-quiet* OR *\\wscript.exe\ *.jse OR *\\wscript.exe\ *.js OR *\\wscript.exe\ *.vba OR *\\wscript.exe\ *.vbe OR *\\cscript.exe\ *.jse OR *\\cscript.exe\ *.js OR *\\cscript.exe\ *.vba OR *\\cscript.exe\ *.vbe OR *\\fodhelper.exe OR *waitfor*\/s* OR *waitfor*\/si\ persist* OR *remote*\/s* OR *remote*\/c* OR *remote*\/q* OR *AddInProcess* OR *\ \/stext\ * OR *\ \/scomma\ * OR *\ \/stab\ * OR *\ \/stabular\ * OR *\ \/shtml\ * OR *\ \/sverhtml\ * OR *\ \/sxml\ *)
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/5f0f47a5-cb16-4dbe-9e31-e8d976d73de3 <<EOF
{
  "metadata": {
    "title": "Suspicious Process Creation",
    "description": "Detects suspicious process starts on Windows systems based on keywords",
    "tags": [
      "car.2013-07-001"
    ],
    "query": "winlog.event_data.CommandLine.keyword:(*\\ sekurlsa\\:* OR net\\ localgroup\\ administrators\\ *\\ \\/add OR net\\ group\\ \\\"Domain\\ Admins\\\"\\ *\\ \\/ADD\\ \\/DOMAIN OR certutil.exe\\ *\\-urlcache*\\ http* OR certutil.exe\\ *\\-urlcache*\\ ftp* OR netsh\\ advfirewall\\ firewall\\ *\\\\AppData\\\\* OR attrib\\ \\+S\\ \\+H\\ \\+R\\ *\\\\AppData\\\\* OR schtasks*\\ \\/create\\ *\\\\AppData\\\\* OR schtasks*\\ \\/sc\\ minute* OR *\\\\Regasm.exe\\ *\\\\AppData\\\\* OR *\\\\Regasm\\ *\\\\AppData\\\\* OR *\\\\bitsadmin*\\ \\/transfer* OR *\\\\certutil.exe\\ *\\ \\-decode\\ * OR *\\\\certutil.exe\\ *\\ \\-decodehex\\ * OR *\\\\certutil.exe\\ \\-ping\\ * OR icacls\\ *\\ \\/grant\\ Everyone\\:F\\ \\/T\\ \\/C\\ \\/Q OR *\\ wbadmin.exe\\ delete\\ catalog\\ \\-quiet* OR *\\\\wscript.exe\\ *.jse OR *\\\\wscript.exe\\ *.js OR *\\\\wscript.exe\\ *.vba OR *\\\\wscript.exe\\ *.vbe OR *\\\\cscript.exe\\ *.jse OR *\\\\cscript.exe\\ *.js OR *\\\\cscript.exe\\ *.vba OR *\\\\cscript.exe\\ *.vbe OR *\\\\fodhelper.exe OR *waitfor*\\/s* OR *waitfor*\\/si\\ persist* OR *remote*\\/s* OR *remote*\\/c* OR *remote*\\/q* OR *AddInProcess* OR *\\ \\/stext\\ * OR *\\ \\/scomma\\ * OR *\\ \\/stab\\ * OR *\\ \\/stabular\\ * OR *\\ \\/shtml\\ * OR *\\ \\/sverhtml\\ * OR *\\ \\/sxml\\ *)"
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
                    "query": "winlog.event_data.CommandLine.keyword:(*\\ sekurlsa\\:* OR net\\ localgroup\\ administrators\\ *\\ \\/add OR net\\ group\\ \\\"Domain\\ Admins\\\"\\ *\\ \\/ADD\\ \\/DOMAIN OR certutil.exe\\ *\\-urlcache*\\ http* OR certutil.exe\\ *\\-urlcache*\\ ftp* OR netsh\\ advfirewall\\ firewall\\ *\\\\AppData\\\\* OR attrib\\ \\+S\\ \\+H\\ \\+R\\ *\\\\AppData\\\\* OR schtasks*\\ \\/create\\ *\\\\AppData\\\\* OR schtasks*\\ \\/sc\\ minute* OR *\\\\Regasm.exe\\ *\\\\AppData\\\\* OR *\\\\Regasm\\ *\\\\AppData\\\\* OR *\\\\bitsadmin*\\ \\/transfer* OR *\\\\certutil.exe\\ *\\ \\-decode\\ * OR *\\\\certutil.exe\\ *\\ \\-decodehex\\ * OR *\\\\certutil.exe\\ \\-ping\\ * OR icacls\\ *\\ \\/grant\\ Everyone\\:F\\ \\/T\\ \\/C\\ \\/Q OR *\\ wbadmin.exe\\ delete\\ catalog\\ \\-quiet* OR *\\\\wscript.exe\\ *.jse OR *\\\\wscript.exe\\ *.js OR *\\\\wscript.exe\\ *.vba OR *\\\\wscript.exe\\ *.vbe OR *\\\\cscript.exe\\ *.jse OR *\\\\cscript.exe\\ *.js OR *\\\\cscript.exe\\ *.vba OR *\\\\cscript.exe\\ *.vbe OR *\\\\fodhelper.exe OR *waitfor*\\/s* OR *waitfor*\\/si\\ persist* OR *remote*\\/s* OR *remote*\\/c* OR *remote*\\/q* OR *AddInProcess* OR *\\ \\/stext\\ * OR *\\ \\/scomma\\ * OR *\\ \\/stab\\ * OR *\\ \\/stabular\\ * OR *\\ \\/shtml\\ * OR *\\ \\/sverhtml\\ * OR *\\ \\/sxml\\ *)",
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
        "subject": "Sigma Rule 'Suspicious Process Creation'",
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
CommandLine.keyword:(* sekurlsa\:* net localgroup administrators * \/add net group \"Domain Admins\" * \/ADD \/DOMAIN certutil.exe *\-urlcache* http* certutil.exe *\-urlcache* ftp* netsh advfirewall firewall *\\AppData\\* attrib \+S \+H \+R *\\AppData\\* schtasks* \/create *\\AppData\\* schtasks* \/sc minute* *\\Regasm.exe *\\AppData\\* *\\Regasm *\\AppData\\* *\\bitsadmin* \/transfer* *\\certutil.exe * \-decode * *\\certutil.exe * \-decodehex * *\\certutil.exe \-ping * icacls * \/grant Everyone\:F \/T \/C \/Q * wbadmin.exe delete catalog \-quiet* *\\wscript.exe *.jse *\\wscript.exe *.js *\\wscript.exe *.vba *\\wscript.exe *.vbe *\\cscript.exe *.jse *\\cscript.exe *.js *\\cscript.exe *.vba *\\cscript.exe *.vbe *\\fodhelper.exe *waitfor*\/s* *waitfor*\/si persist* *remote*\/s* *remote*\/c* *remote*\/q* *AddInProcess* * \/stext * * \/scomma * * \/stab * * \/stabular * * \/shtml * * \/sverhtml * * \/sxml *)
```


### splunk
    
```
(CommandLine="* sekurlsa:*" OR CommandLine="net localgroup administrators * /add" OR CommandLine="net group \"Domain Admins\" * /ADD /DOMAIN" OR CommandLine="certutil.exe *-urlcache* http*" OR CommandLine="certutil.exe *-urlcache* ftp*" OR CommandLine="netsh advfirewall firewall *\\AppData\\*" OR CommandLine="attrib +S +H +R *\\AppData\\*" OR CommandLine="schtasks* /create *\\AppData\\*" OR CommandLine="schtasks* /sc minute*" OR CommandLine="*\\Regasm.exe *\\AppData\\*" OR CommandLine="*\\Regasm *\\AppData\\*" OR CommandLine="*\\bitsadmin* /transfer*" OR CommandLine="*\\certutil.exe * -decode *" OR CommandLine="*\\certutil.exe * -decodehex *" OR CommandLine="*\\certutil.exe -ping *" OR CommandLine="icacls * /grant Everyone:F /T /C /Q" OR CommandLine="* wbadmin.exe delete catalog -quiet*" OR CommandLine="*\\wscript.exe *.jse" OR CommandLine="*\\wscript.exe *.js" OR CommandLine="*\\wscript.exe *.vba" OR CommandLine="*\\wscript.exe *.vbe" OR CommandLine="*\\cscript.exe *.jse" OR CommandLine="*\\cscript.exe *.js" OR CommandLine="*\\cscript.exe *.vba" OR CommandLine="*\\cscript.exe *.vbe" OR CommandLine="*\\fodhelper.exe" OR CommandLine="*waitfor*/s*" OR CommandLine="*waitfor*/si persist*" OR CommandLine="*remote*/s*" OR CommandLine="*remote*/c*" OR CommandLine="*remote*/q*" OR CommandLine="*AddInProcess*" OR CommandLine="* /stext *" OR CommandLine="* /scomma *" OR CommandLine="* /stab *" OR CommandLine="* /stabular *" OR CommandLine="* /shtml *" OR CommandLine="* /sverhtml *" OR CommandLine="* /sxml *") | table ComputerName,User,CommandLine
```


### logpoint
    
```
CommandLine IN ["* sekurlsa:*", "net localgroup administrators * /add", "net group \"Domain Admins\" * /ADD /DOMAIN", "certutil.exe *-urlcache* http*", "certutil.exe *-urlcache* ftp*", "netsh advfirewall firewall *\\AppData\\*", "attrib +S +H +R *\\AppData\\*", "schtasks* /create *\\AppData\\*", "schtasks* /sc minute*", "*\\Regasm.exe *\\AppData\\*", "*\\Regasm *\\AppData\\*", "*\\bitsadmin* /transfer*", "*\\certutil.exe * -decode *", "*\\certutil.exe * -decodehex *", "*\\certutil.exe -ping *", "icacls * /grant Everyone:F /T /C /Q", "* wbadmin.exe delete catalog -quiet*", "*\\wscript.exe *.jse", "*\\wscript.exe *.js", "*\\wscript.exe *.vba", "*\\wscript.exe *.vbe", "*\\cscript.exe *.jse", "*\\cscript.exe *.js", "*\\cscript.exe *.vba", "*\\cscript.exe *.vbe", "*\\fodhelper.exe", "*waitfor*/s*", "*waitfor*/si persist*", "*remote*/s*", "*remote*/c*", "*remote*/q*", "*AddInProcess*", "* /stext *", "* /scomma *", "* /stab *", "* /stabular *", "* /shtml *", "* /sverhtml *", "* /sxml *"]
```


### grep
    
```
grep -P '^(?:.*.* sekurlsa:.*|.*net localgroup administrators .* /add|.*net group "Domain Admins" .* /ADD /DOMAIN|.*certutil\.exe .*-urlcache.* http.*|.*certutil\.exe .*-urlcache.* ftp.*|.*netsh advfirewall firewall .*\AppData\\.*|.*attrib \+S \+H \+R .*\AppData\\.*|.*schtasks.* /create .*\AppData\\.*|.*schtasks.* /sc minute.*|.*.*\Regasm\.exe .*\AppData\\.*|.*.*\Regasm .*\AppData\\.*|.*.*\bitsadmin.* /transfer.*|.*.*\certutil\.exe .* -decode .*|.*.*\certutil\.exe .* -decodehex .*|.*.*\certutil\.exe -ping .*|.*icacls .* /grant Everyone:F /T /C /Q|.*.* wbadmin\.exe delete catalog -quiet.*|.*.*\wscript\.exe .*\.jse|.*.*\wscript\.exe .*\.js|.*.*\wscript\.exe .*\.vba|.*.*\wscript\.exe .*\.vbe|.*.*\cscript\.exe .*\.jse|.*.*\cscript\.exe .*\.js|.*.*\cscript\.exe .*\.vba|.*.*\cscript\.exe .*\.vbe|.*.*\fodhelper\.exe|.*.*waitfor.*/s.*|.*.*waitfor.*/si persist.*|.*.*remote.*/s.*|.*.*remote.*/c.*|.*.*remote.*/q.*|.*.*AddInProcess.*|.*.* /stext .*|.*.* /scomma .*|.*.* /stab .*|.*.* /stabular .*|.*.* /shtml .*|.*.* /sverhtml .*|.*.* /sxml .*)'
```



