```
| rest /servicesNS/-/-/saved/searches search="is_scheduled=1" search="disabled=0"  
| regex cron_schedule="((\*\/[0-5])|(\*)|(\d+\-\d+\/[0-5])) (\*) (\*) (\*) (\*)"
| rename eai:acl.app as app, title as savedsearch_name, eai:acl.owner as owner, eai:acl.sharing as sharing
| search NOT (app="splunk_monitoring_console" AND savedsearch_name="DMC Asset - Build Standalone Asset Table")
| eval last_seen=  now()
| eval current = 1
| append [inputlookup append=t scheduled_searches | search current!="missing"| fields - current]
| eval known = case(isnull(current), 1)
| eval scheduled_type=if(alert_type=="always", "report", "alert")
| lookup splunkref fqdn as splunk_server OUTPUT function
| stats values(function) as function  first(current) as current first(known) as known, first(cron_schedule) as cron_schedule, first(scheduled_type) as  scheduled_type by app,savedsearch_name,author
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
| outputlookup scheduled_searches
| fields - known current
| search change=*
```
