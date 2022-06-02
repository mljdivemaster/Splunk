```
| rest /servicesNS/-/-/saved/searches search="is_scheduled=1" search="disabled=0"  
| rename eai:acl.app as app, title as savedsearch_name, eai:acl.owner as owner, eai:acl.sharing as sharing
```Search for only alerts```
```| search alert_type!="always"```
| search NOT (app="splunk_instrumentation" AND savedsearch_name="instrumentation.usage.*")
| eval last_seen=  now()
| eval current = 1
| append [inputlookup append=t known_alert_searches.csv | search current!="missing"| fields - current]
| eval known = case(isnull(current), 1)
| eval scheduled_type=if(alert_type=="always", "report", "alert")
| lookup splunkref fqdn as splunk_server OUTPUT function
| stats values(function) as function first(current) as current first(known) as known, first(cron_schedule) as cron_schedule, first(scheduled_type) as  scheduled_type by app,savedsearch_name,author
| sort cron_schedule
| fillnull value="missing" current
| fillnull value="new" known
| eval change=case(
current=="missing",current,
known=="new",known)
``` Following is additional for ITSI. It translates any base search id's found in savedsearch_name with ITSI KPI Base search name. Thanks to Sara Lord for supplying newly baked bread that helped to solve it!```
| rex field=savedsearch_name "Indicator - Shared - (?<key>.+?)\s-"
| join [| inputlookup kpi_base_search_title_lookup | eval key=_key]
| rename title as kpi_base_search_title
|fields - key
| outputlookup known_alert_searches.csv
| search change=*
| fields - known current
```
