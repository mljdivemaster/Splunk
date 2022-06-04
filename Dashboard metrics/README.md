# Dashboard metrics
This user case is to identify where creativity amongst your dashboards. It shows different aspects of viewed or modified dashboards. It is based on event types and scheduled reports in order to extract information located in the internal index, to make the public available.

## Event types
<p>This is a way to label and tag your data, to explain what each event actual means. It works as a glue between the raw data and a human friendly presentation of the data. <br>
Create a new event type by selecting the meny "Settings", "Event types", and click on "New event type". <br>
Following event types needs to be created for the report to work. </p>

### splunk_dashboard_update
Used by the report:

Name:
`splunk_dashboard_update`

Search String:
`index=_internal sourcetype=splunkd_ui_access method=POST  file="*"  uri="*/data/ui/views/*" NOT uri=*/acl status=200`

Tags:
`dashboard splunk update`
### splunk_dashboard_acl
Not used anywhere yet.

Name:
`splunk_dashboard_acl`

Search string:
`index=_internal sourcetype=splunkd_ui_access method=POST uri="*/data/ui/views/*/acl" status=200`

Tags:
`acl dashboard splunk`

### splunk_dashboard_view
Used by the report:

Name:
`splunk_dashboard_view`

Search string:
`index=_internal sourcetype=splunk_web_access method=GET status=200 app=* view=*`

Tags:
`dashboard splunk view`

## Field extractions
I have created some field extractions in order to have the app and dashboard in separate fields.

Name:
```
AppName,ViewName```

Apply to sourcetype:
```splunkd_ui_access```

Extraction/Transform:

```	\/app\/(?<AppName>\w+)\/(?<ViewName>\w+)```

## Reports
### Most Used Dashboards with Details
```
index=_internal sourcetype=splunkd_ui_access uri_path=*/app/* user!="-" AppName=* ViewName=* ViewName!=contents ViewName!=search AppName!=Launcher
| stats latest(_time) AS latest_access_time count  dc(user) as user by ViewName AppName
| convert ctime(latest_access_time)
| sort - count dashboard
| eval App=AppName
| rename ViewName as "Dashboard Name" user as User  latest_access_time as "Latest View" count as Hits| table App, "Latest View", Hits, User, "Dashboard Name"
```

### Updated Dashboards stats - Last 7 Days
```
|  union maxtime=600 timeout=3000
    [search eventtype="splunk_dashboard_update"
    |  stats count as c, dc(app) as dc_apps, dc(dashboard) as dc_dashboards, dc(user) as dc_users]
    [search eventtype="splunk_dashboard_update"
    |  top user
    |   rename count as user_count]
    [search eventtype="splunk_dashboard_update"
    |  rename dashboard as dashboards
    |  top dashboards
    |  rename count as dashboard_count ]
    [search eventtype="splunk_dashboard_update"
    |  rename app as app_c
    |  top app_c
    |  rename count as app_count ]
    [search eventtype="splunk_dashboard_update"
    |  rename app as apps, dashboard as dc_dashboard
    |  stats dc(user) as colab by dc_dashboard apps
    |  search colab>1]
    [search eventtype="splunk_dashboard_update"
    |  table _time app dashboard
    |  dedup app dashboard
    |  sort - _time]
    |  fields - percent
    ```
