| Title                    | Possible DNS Rebinding       |
|:-------------------------|:------------------|
| **Description**          | Detects several different DNS-answers by one domain with IPs from internal and external networks. Normally, DNS-answer contain TTL >100. (DNS-record will saved in host cache for a while TTL). |
| **ATT&amp;CK Tactic**    |  <ul><li>[TA0001: Initial Access](https://attack.mitre.org/tactics/TA0001)</li></ul>  |
| **ATT&amp;CK Technique** | <ul><li>[T1189: Drive-by Compromise](https://attack.mitre.org/techniques/T1189)</li></ul>  |
| **Data Needed**          | <ul><li>[DN_0085_22_windows_sysmon_DnsQuery](../Data_Needed/DN_0085_22_windows_sysmon_DnsQuery.md)</li></ul>  |
| **Trigger**              |  There is no documented Trigger for this Detection Rule yet  |
| **Severity Level**       | medium |
| **False Positives**      |  There are no documented False Positives for this Detection Rule yet  |
| **Development Status**   | experimental |
| **References**           | <ul><li>[https://medium.com/@brannondorsey/attacking-private-networks-from-the-internet-with-dns-rebinding-ea7098a2d325](https://medium.com/@brannondorsey/attacking-private-networks-from-the-internet-with-dns-rebinding-ea7098a2d325)</li></ul>  |
| **Author**               | Ilyas Ochkov, oscd.community |


## Detection Rules

### Sigma rule

```
title: Possible DNS Rebinding
id: eb07e747-2552-44cd-af36-b659ae0958e4
status: experimental
description: Detects several different DNS-answers by one domain with IPs from internal and external networks. Normally, DNS-answer contain TTL >100. (DNS-record will saved in host cache for a while TTL).
date: 2019/10/25
modified: 2020/08/28
author: Ilyas Ochkov, oscd.community
references:
    - https://medium.com/@brannondorsey/attacking-private-networks-from-the-internet-with-dns-rebinding-ea7098a2d325
tags:
    - attack.initial_access
    - attack.t1189
logsource:
    product: windows
    service: sysmon
detection:
    dns_answer:
        EventID: 22
        QueryName: '*'
        QueryStatus: '0'
    filter_int_ip:
        QueryResults|startswith:
            - '(::ffff:)?10.'
            - '(::ffff:)?192.168.'
            - '(::ffff:)?172.16.'
            - '(::ffff:)?172.17.'
            - '(::ffff:)?172.18.'
            - '(::ffff:)?172.19.'
            - '(::ffff:)?172.20.'
            - '(::ffff:)?172.21.'
            - '(::ffff:)?172.22.'
            - '(::ffff:)?172.23.'
            - '(::ffff:)?172.24.'
            - '(::ffff:)?172.25.'
            - '(::ffff:)?172.26.'
            - '(::ffff:)?172.27.'
            - '(::ffff:)?172.28.'
            - '(::ffff:)?172.29.'
            - '(::ffff:)?172.30.'
            - '(::ffff:)?172.31.'
            - '(::ffff:)?127.'
    timeframe: 30s
    condition: (dns_answer and filter_int_ip) and (dns_answer and not filter_int_ip) | count(QueryName) by ComputerName > 3
level: medium

```





### powershell
    
```
Get-WinEvent -LogName Microsoft-Windows-Sysmon/Operational | where {($_.ID -eq "22" -and $_.message -match "QueryName.*.*" -and $_.message -match "QueryStatus.*0" -and ($_.message -match "QueryResults.*(::ffff:)?10..*" -or $_.message -match "QueryResults.*(::ffff:)?192.168..*" -or $_.message -match "QueryResults.*(::ffff:)?172.16..*" -or $_.message -match "QueryResults.*(::ffff:)?172.17..*" -or $_.message -match "QueryResults.*(::ffff:)?172.18..*" -or $_.message -match "QueryResults.*(::ffff:)?172.19..*" -or $_.message -match "QueryResults.*(::ffff:)?172.20..*" -or $_.message -match "QueryResults.*(::ffff:)?172.21..*" -or $_.message -match "QueryResults.*(::ffff:)?172.22..*" -or $_.message -match "QueryResults.*(::ffff:)?172.23..*" -or $_.message -match "QueryResults.*(::ffff:)?172.24..*" -or $_.message -match "QueryResults.*(::ffff:)?172.25..*" -or $_.message -match "QueryResults.*(::ffff:)?172.26..*" -or $_.message -match "QueryResults.*(::ffff:)?172.27..*" -or $_.message -match "QueryResults.*(::ffff:)?172.28..*" -or $_.message -match "QueryResults.*(::ffff:)?172.29..*" -or $_.message -match "QueryResults.*(::ffff:)?172.30..*" -or $_.message -match "QueryResults.*(::ffff:)?172.31..*" -or $_.message -match "QueryResults.*(::ffff:)?127..*") -and ($_.ID -eq "22" -and $_.message -match "QueryName.*.*" -and $_.message -match "QueryStatus.*0") -and  -not (($_.message -match "QueryResults.*(::ffff:)?10..*" -or $_.message -match "QueryResults.*(::ffff:)?192.168..*" -or $_.message -match "QueryResults.*(::ffff:)?172.16..*" -or $_.message -match "QueryResults.*(::ffff:)?172.17..*" -or $_.message -match "QueryResults.*(::ffff:)?172.18..*" -or $_.message -match "QueryResults.*(::ffff:)?172.19..*" -or $_.message -match "QueryResults.*(::ffff:)?172.20..*" -or $_.message -match "QueryResults.*(::ffff:)?172.21..*" -or $_.message -match "QueryResults.*(::ffff:)?172.22..*" -or $_.message -match "QueryResults.*(::ffff:)?172.23..*" -or $_.message -match "QueryResults.*(::ffff:)?172.24..*" -or $_.message -match "QueryResults.*(::ffff:)?172.25..*" -or $_.message -match "QueryResults.*(::ffff:)?172.26..*" -or $_.message -match "QueryResults.*(::ffff:)?172.27..*" -or $_.message -match "QueryResults.*(::ffff:)?172.28..*" -or $_.message -match "QueryResults.*(::ffff:)?172.29..*" -or $_.message -match "QueryResults.*(::ffff:)?172.30..*" -or $_.message -match "QueryResults.*(::ffff:)?172.31..*" -or $_.message -match "QueryResults.*(::ffff:)?127..*"))) }  | select ComputerName, QueryName | group ComputerName | foreach { [PSCustomObject]@{'ComputerName'=$_.name;'Count'=($_.group.QueryName | sort -u).count} }  | sort count -desc | where { $_.count -gt 3 }
```


### es-qs
    
```
An unsupported feature is required for this Sigma rule (detection_rules/sigma/rules/windows/sysmon/sysmon_possible_dns_rebinding.yml): Aggregations not implemented for this backend
Feel free to contribute for fun and fame, this is open source :) -> https://github.com/Neo23x0/sigma
```


### xpack-watcher
    
```
curl -s -XPUT -H 'Content-Type: application/json' --data-binary @- localhost:9200/_watcher/watch/eb07e747-2552-44cd-af36-b659ae0958e4 <<EOF
{
  "metadata": {
    "title": "Possible DNS Rebinding",
    "description": "Detects several different DNS-answers by one domain with IPs from internal and external networks. Normally, DNS-answer contain TTL >100. (DNS-record will saved in host cache for a while TTL).",
    "tags": [
      "attack.initial_access",
      "attack.t1189"
    ],
    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"22\" AND QueryName.keyword:* AND QueryStatus:\"0\" AND QueryResults.keyword:(\\(\\:\\:ffff\\:\\)?10.* OR \\(\\:\\:ffff\\:\\)?192.168.* OR \\(\\:\\:ffff\\:\\)?172.16.* OR \\(\\:\\:ffff\\:\\)?172.17.* OR \\(\\:\\:ffff\\:\\)?172.18.* OR \\(\\:\\:ffff\\:\\)?172.19.* OR \\(\\:\\:ffff\\:\\)?172.20.* OR \\(\\:\\:ffff\\:\\)?172.21.* OR \\(\\:\\:ffff\\:\\)?172.22.* OR \\(\\:\\:ffff\\:\\)?172.23.* OR \\(\\:\\:ffff\\:\\)?172.24.* OR \\(\\:\\:ffff\\:\\)?172.25.* OR \\(\\:\\:ffff\\:\\)?172.26.* OR \\(\\:\\:ffff\\:\\)?172.27.* OR \\(\\:\\:ffff\\:\\)?172.28.* OR \\(\\:\\:ffff\\:\\)?172.29.* OR \\(\\:\\:ffff\\:\\)?172.30.* OR \\(\\:\\:ffff\\:\\)?172.31.* OR \\(\\:\\:ffff\\:\\)?127.*) AND (winlog.event_id:\"22\" AND QueryName.keyword:* AND QueryStatus:\"0\") AND (NOT (QueryResults.keyword:(\\(\\:\\:ffff\\:\\)?10.* OR \\(\\:\\:ffff\\:\\)?192.168.* OR \\(\\:\\:ffff\\:\\)?172.16.* OR \\(\\:\\:ffff\\:\\)?172.17.* OR \\(\\:\\:ffff\\:\\)?172.18.* OR \\(\\:\\:ffff\\:\\)?172.19.* OR \\(\\:\\:ffff\\:\\)?172.20.* OR \\(\\:\\:ffff\\:\\)?172.21.* OR \\(\\:\\:ffff\\:\\)?172.22.* OR \\(\\:\\:ffff\\:\\)?172.23.* OR \\(\\:\\:ffff\\:\\)?172.24.* OR \\(\\:\\:ffff\\:\\)?172.25.* OR \\(\\:\\:ffff\\:\\)?172.26.* OR \\(\\:\\:ffff\\:\\)?172.27.* OR \\(\\:\\:ffff\\:\\)?172.28.* OR \\(\\:\\:ffff\\:\\)?172.29.* OR \\(\\:\\:ffff\\:\\)?172.30.* OR \\(\\:\\:ffff\\:\\)?172.31.* OR \\(\\:\\:ffff\\:\\)?127.*))))"
  },
  "trigger": {
    "schedule": {
      "interval": "30s"
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
                    "query": "(winlog.channel:\"Microsoft\\-Windows\\-Sysmon\\/Operational\" AND winlog.event_id:\"22\" AND QueryName.keyword:* AND QueryStatus:\"0\" AND QueryResults.keyword:(\\(\\:\\:ffff\\:\\)?10.* OR \\(\\:\\:ffff\\:\\)?192.168.* OR \\(\\:\\:ffff\\:\\)?172.16.* OR \\(\\:\\:ffff\\:\\)?172.17.* OR \\(\\:\\:ffff\\:\\)?172.18.* OR \\(\\:\\:ffff\\:\\)?172.19.* OR \\(\\:\\:ffff\\:\\)?172.20.* OR \\(\\:\\:ffff\\:\\)?172.21.* OR \\(\\:\\:ffff\\:\\)?172.22.* OR \\(\\:\\:ffff\\:\\)?172.23.* OR \\(\\:\\:ffff\\:\\)?172.24.* OR \\(\\:\\:ffff\\:\\)?172.25.* OR \\(\\:\\:ffff\\:\\)?172.26.* OR \\(\\:\\:ffff\\:\\)?172.27.* OR \\(\\:\\:ffff\\:\\)?172.28.* OR \\(\\:\\:ffff\\:\\)?172.29.* OR \\(\\:\\:ffff\\:\\)?172.30.* OR \\(\\:\\:ffff\\:\\)?172.31.* OR \\(\\:\\:ffff\\:\\)?127.*) AND (winlog.event_id:\"22\" AND QueryName.keyword:* AND QueryStatus:\"0\") AND (NOT (QueryResults.keyword:(\\(\\:\\:ffff\\:\\)?10.* OR \\(\\:\\:ffff\\:\\)?192.168.* OR \\(\\:\\:ffff\\:\\)?172.16.* OR \\(\\:\\:ffff\\:\\)?172.17.* OR \\(\\:\\:ffff\\:\\)?172.18.* OR \\(\\:\\:ffff\\:\\)?172.19.* OR \\(\\:\\:ffff\\:\\)?172.20.* OR \\(\\:\\:ffff\\:\\)?172.21.* OR \\(\\:\\:ffff\\:\\)?172.22.* OR \\(\\:\\:ffff\\:\\)?172.23.* OR \\(\\:\\:ffff\\:\\)?172.24.* OR \\(\\:\\:ffff\\:\\)?172.25.* OR \\(\\:\\:ffff\\:\\)?172.26.* OR \\(\\:\\:ffff\\:\\)?172.27.* OR \\(\\:\\:ffff\\:\\)?172.28.* OR \\(\\:\\:ffff\\:\\)?172.29.* OR \\(\\:\\:ffff\\:\\)?172.30.* OR \\(\\:\\:ffff\\:\\)?172.31.* OR \\(\\:\\:ffff\\:\\)?127.*))))",
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
          },
          "aggs": {
            "by": {
              "terms": {
                "field": "winlog.ComputerName",
                "size": 10,
                "order": {
                  "_count": "desc"
                },
                "min_doc_count": 4
              },
              "aggs": {
                "agg": {
                  "terms": {
                    "field": "QueryName",
                    "size": 10,
                    "order": {
                      "_count": "desc"
                    },
                    "min_doc_count": 4
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
      "ctx.payload.aggregations.by.buckets.0.agg.buckets.0.doc_count": {
        "gt": 3
      }
    }
  },
  "actions": {
    "send_email": {
      "throttle_period": "15m",
      "email": {
        "profile": "standard",
        "from": "root@localhost",
        "to": "root@localhost",
        "subject": "Sigma Rule 'Possible DNS Rebinding'",
        "body": "Hits:\n{{#aggregations.agg.buckets}}\n {{key}} {{doc_count}}\n\n{{#by.buckets}}\n-- {{key}} {{doc_count}}\n{{/by.buckets}}\n\n{{/aggregations.agg.buckets}}\n",
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
An unsupported feature is required for this Sigma rule (detection_rules/sigma/rules/windows/sysmon/sysmon_possible_dns_rebinding.yml): Aggregations not implemented for this backend
Feel free to contribute for fun and fame, this is open source :) -> https://github.com/Neo23x0/sigma
```


### splunk
    
```
(source="WinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode="22" QueryName="*" QueryStatus="0" (QueryResults="(::ffff:)?10.*" OR QueryResults="(::ffff:)?192.168.*" OR QueryResults="(::ffff:)?172.16.*" OR QueryResults="(::ffff:)?172.17.*" OR QueryResults="(::ffff:)?172.18.*" OR QueryResults="(::ffff:)?172.19.*" OR QueryResults="(::ffff:)?172.20.*" OR QueryResults="(::ffff:)?172.21.*" OR QueryResults="(::ffff:)?172.22.*" OR QueryResults="(::ffff:)?172.23.*" OR QueryResults="(::ffff:)?172.24.*" OR QueryResults="(::ffff:)?172.25.*" OR QueryResults="(::ffff:)?172.26.*" OR QueryResults="(::ffff:)?172.27.*" OR QueryResults="(::ffff:)?172.28.*" OR QueryResults="(::ffff:)?172.29.*" OR QueryResults="(::ffff:)?172.30.*" OR QueryResults="(::ffff:)?172.31.*" OR QueryResults="(::ffff:)?127.*") (EventCode="22" QueryName="*" QueryStatus="0") NOT ((QueryResults="(::ffff:)?10.*" OR QueryResults="(::ffff:)?192.168.*" OR QueryResults="(::ffff:)?172.16.*" OR QueryResults="(::ffff:)?172.17.*" OR QueryResults="(::ffff:)?172.18.*" OR QueryResults="(::ffff:)?172.19.*" OR QueryResults="(::ffff:)?172.20.*" OR QueryResults="(::ffff:)?172.21.*" OR QueryResults="(::ffff:)?172.22.*" OR QueryResults="(::ffff:)?172.23.*" OR QueryResults="(::ffff:)?172.24.*" OR QueryResults="(::ffff:)?172.25.*" OR QueryResults="(::ffff:)?172.26.*" OR QueryResults="(::ffff:)?172.27.*" OR QueryResults="(::ffff:)?172.28.*" OR QueryResults="(::ffff:)?172.29.*" OR QueryResults="(::ffff:)?172.30.*" OR QueryResults="(::ffff:)?172.31.*" OR QueryResults="(::ffff:)?127.*"))) | eventstats dc(QueryName) as val by ComputerName | search val > 3
```


### logpoint
    
```
(event_id="22" QueryName="*" QueryStatus="0" QueryResults IN ["(::ffff:)?10.*", "(::ffff:)?192.168.*", "(::ffff:)?172.16.*", "(::ffff:)?172.17.*", "(::ffff:)?172.18.*", "(::ffff:)?172.19.*", "(::ffff:)?172.20.*", "(::ffff:)?172.21.*", "(::ffff:)?172.22.*", "(::ffff:)?172.23.*", "(::ffff:)?172.24.*", "(::ffff:)?172.25.*", "(::ffff:)?172.26.*", "(::ffff:)?172.27.*", "(::ffff:)?172.28.*", "(::ffff:)?172.29.*", "(::ffff:)?172.30.*", "(::ffff:)?172.31.*", "(::ffff:)?127.*"] (event_id="22" QueryName="*" QueryStatus="0")  -(QueryResults IN ["(::ffff:)?10.*", "(::ffff:)?192.168.*", "(::ffff:)?172.16.*", "(::ffff:)?172.17.*", "(::ffff:)?172.18.*", "(::ffff:)?172.19.*", "(::ffff:)?172.20.*", "(::ffff:)?172.21.*", "(::ffff:)?172.22.*", "(::ffff:)?172.23.*", "(::ffff:)?172.24.*", "(::ffff:)?172.25.*", "(::ffff:)?172.26.*", "(::ffff:)?172.27.*", "(::ffff:)?172.28.*", "(::ffff:)?172.29.*", "(::ffff:)?172.30.*", "(::ffff:)?172.31.*", "(::ffff:)?127.*"])) | chart count(QueryName) as val by ComputerName | search val > 3
```


### grep
    
```
grep -P '^(?:.*(?=.*22)(?=.*.*)(?=.*0)(?=.*(?:.*\(::ffff:\)?10\..*|.*\(::ffff:\)?192\.168\..*|.*\(::ffff:\)?172\.16\..*|.*\(::ffff:\)?172\.17\..*|.*\(::ffff:\)?172\.18\..*|.*\(::ffff:\)?172\.19\..*|.*\(::ffff:\)?172\.20\..*|.*\(::ffff:\)?172\.21\..*|.*\(::ffff:\)?172\.22\..*|.*\(::ffff:\)?172\.23\..*|.*\(::ffff:\)?172\.24\..*|.*\(::ffff:\)?172\.25\..*|.*\(::ffff:\)?172\.26\..*|.*\(::ffff:\)?172\.27\..*|.*\(::ffff:\)?172\.28\..*|.*\(::ffff:\)?172\.29\..*|.*\(::ffff:\)?172\.30\..*|.*\(::ffff:\)?172\.31\..*|.*\(::ffff:\)?127\..*))(?=.*(?:.*(?=.*22)(?=.*.*)(?=.*0)))(?=.*(?!.*(?:.*(?=.*(?:.*\(::ffff:\)?10\..*|.*\(::ffff:\)?192\.168\..*|.*\(::ffff:\)?172\.16\..*|.*\(::ffff:\)?172\.17\..*|.*\(::ffff:\)?172\.18\..*|.*\(::ffff:\)?172\.19\..*|.*\(::ffff:\)?172\.20\..*|.*\(::ffff:\)?172\.21\..*|.*\(::ffff:\)?172\.22\..*|.*\(::ffff:\)?172\.23\..*|.*\(::ffff:\)?172\.24\..*|.*\(::ffff:\)?172\.25\..*|.*\(::ffff:\)?172\.26\..*|.*\(::ffff:\)?172\.27\..*|.*\(::ffff:\)?172\.28\..*|.*\(::ffff:\)?172\.29\..*|.*\(::ffff:\)?172\.30\..*|.*\(::ffff:\)?172\.31\..*|.*\(::ffff:\)?127\..*))))))'
```



