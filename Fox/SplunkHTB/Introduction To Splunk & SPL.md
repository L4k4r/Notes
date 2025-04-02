
**Find through an SPL search against all data the account name with the highest amount of Kerberos authentication ticket requests.** 

First check all the different sourcetypes
```
index="main" | stats count by sourcetype
```

```
WinEventLog:Application 9196 
WinEventLog:Security 68568 
WinEventLog:Sysmon 358108 
WinEventLog:System 8517 
linux:auth 208 
linux:syslog 120719
```

The Kerberos authentication ticket request has an EventID of 4768. This eventcode is typically a Windows Security Log event.

We can verify this by:
```
index="main" EventCode=4768 | stats count by sourcetype

WinEventLog:Security	48
```

So to get the answer:

```
index=* sourcetype="WinEventLog:Security" EventCode=4768 
| stats count by Account_Name 
| sort - count 
| head 1
```

Where: 
	- `index=*` : Searches across all indexes.
	- `sourcetpye="WinEventLog:Security` : Filters the search to only include events with the specific source type.
	- `EventCode=4768`: Filters events to only include those with eventcode 4768, which corresponds to Kerberos TGT requests.
	- `| stats count by Account_Name`: Count the number of events for each unique account name.
	- `| sort - count`: Sorts the results in descending order by count.
	- `| head 1`: Retrieves the top result, which is the name with the highest number of events.

---

**Find through an SPL search against all 4624 events the count of distinct computers accessed by the account name SYSTEM.**

So we want to get all the different computers that the account with the name SYSTEM has accessed.

```
index=* sourcetype="WinEventLog:Security" EventCode=4624 Account_Name="SYSTEM"
| stats dc(ComputerName) as computers
```

So here we filter on the Security event log sourcetype with the eventcode 4624 and the SYSTEM account name. And we count the unique computer names with `dc` (distinct)

```
computers
10
```

----

**Find through an SPL search against all 4624 events the account name that made the most login attempts within a span of 10 minutes.**

```
index=* sourcetype="WinEventLog:Security" EventCode=4624
| stats earliest(_time) as first_login, latest(_time) as last_login, count as login_count by Account_Name
| eval time_diff = last_login - first_login
| where time_diff <= 600
| sort - login_count
| head 1
```

Where:
	 `| stats earliest(_time) as first_login, latest(_time) as last_login, count as login_count by Account_Name` : Groups the events by account name and calculates the first logon time for the account, the last login time for the account and the total number of successful logins for the account.
	 `| evel time_diff = last_login - first_login`: Calculates the difference (in seconds) between the first and last login for each account.
	 `| where time_diff <= 600`: Filters the results to include only accounts where the time difference between the first and last login is less than or equal to 600 seconds.
	 `| sort - login_count`: Sorts the results in descending order of the login count, so the account with the most logins appears first.

```
Account_Name first_login last_login login_count time_diff
aparsa	     1662451859	 1662452062	 9	        203
```


