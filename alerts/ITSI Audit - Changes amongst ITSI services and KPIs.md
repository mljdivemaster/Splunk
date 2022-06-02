```
| inputlookup service_kpi_lookup where object_type="service"
| rename _key as serviceid, title as service_name
| eval kpi_info=mvzip('kpis._key','kpis.title',"==@@==")
| fields + kpi_info, service_name, serviceid
| mvexpand kpi_info
| rex field=kpi_info "(?<kpiid>.+)==@@==(?<kpi_name>.+)"
| eval current = 1
| append [inputlookup append=t service_kpi_lookup.csv | fields - current]
| eval known = case(isnull(current), 1)
| stats  first(current) as current first(known) as known by kpi_info, service_name, serviceid, kpi_name, kpiid
| sort service_name, kpi_name
| outputlookup service_kpi_lookup.csv
|  fillnull value="New entry" known
|  fillnull value="Removed entry" current
|  search known="New entry" OR current="Removed entry"
|  eval changes=if(known=="New entry", "New entry", "Removed entry")
| fields - current known
```
