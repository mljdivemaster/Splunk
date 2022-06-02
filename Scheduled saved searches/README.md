# Scheduled saved searches

This user case is to analyse scheduled saved searches. Which is running the most, longest or skipped a lot. There are several scenarios in the dashboard.

## Lookups
All lookups is preferred to be created in advance, to avoid having them owned by nobody and shared globally. Under "Settings", "Lookups", "Lookup table files", choose "New Lookup Table File".
### Lookup for Splunk infrastructure.
If you have a big environment with generic hostnames, it is good idea to map their function in life to the hostname in this short example:

```
fqdn      function
hostname1 SearchHead Cluster1
hostname2 SearchHead Cluster1
hostname3 SearchHead Cluster1
```
This searches uses a lookup called `splunkref.csv` to limit the search in internal index.

### Lookup for known alerts and reports
 `known_alert_searches.csv` is a lookup to keep track of known scheduled reports and alerts. Used for both the dashboard and the alert that notifies you of added or removed entities.

### Lookup for intensive saved searches
`scheduled_searches.csv` is similar to above but only keeps tracks on intensive alerts and reports. Used for the dashboard mainly.

## Alerts
There are a few alerts that populates some of the lookups, used for the dashboard.

### Splunk Audit - New alerts or scheduled reports
This keeps tracks of all reports and alerts, regardless schedule setting.

### Splunk Audit - New Scheduled searches with interval less than 5 mins
This keeps track of any search that is scheduled every minute, up to every 5 minute.

## Dashboards

### Splunk Audit - Overview of saved searches per user

## Referenses
There is several sources for further reading and knowledge. Start with the excellent presentation from Splunk .conf[ Making the Most
of the Splunk Scheduler ](https://conf.splunk.com/files/2017/slides/making-the-most-of-the-splunk-scheduler.pdf). There is also a Splunk Blog post about this topic [Are you skipping?](https://www.splunk.com/en_us/blog/tips-and-tricks/are-you-skipping-please-read.html)
