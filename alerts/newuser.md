```
index="_internal" sourcetype=splunk_web_access user=*
| eval current="1"
| eval first_seen=_time
| lookup splunkref fqdn as host OUTPUT function
| append
    [inputlookup append=true known_splunk_users.csv
    | fields - current]
| eval known=case(isnull(current), "1")
| append
    [|  rest /services/authentication/users splunk_server=local
    | eval user=title
    | table user realname]
| stats first(current) as current, first(known) as known,  min(first_seen) as first_seen, first(realname) as realname, first(epost) as epost values(function) as function by user
| eval user_add=user+"@your_domain.com"
| eval epost=case(
user LIKE "%@%", user,
1==1, user_add)
| fillnull value="new" known
| outputlookup known_splunk_users.csv
| search known="new"
| eval first_seen=strftime(first_seen, "%Y-%m-%d %H:%M:%S")
```
