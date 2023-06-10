# Splunk ADMIN Exam

1. When using the REST API to access Splunk Cloud, what port is used?
	* **8089**
2. Which node is the cluster label configured on?
	* **Manager Node**
3. Your log events contain user IDs. An external comma-separated values (CSV) file contains user IDs and full names. How would you enhance your search results to include full names?
	* Define a lookup table using the CSV file and use the **lookup** command
4. While planning to scale up to a larger deployment from a single-instance of Splunk Enterprise, **what are the main functions you need to deliver distributed data collection, indexing, and search**?
	* **Index clustering, search head clustering, and a forwarding layer**
5. A log file denotes the start of a process with the string "starting process". It denotes the end of the process with "completed process". You must create a report of transactions that started but never finished. How can you do this?
	* Use the `keepevicted` option with the command `transaction` For example:  `"| transaction startswith
endswith = 'starting process' endswith= 'completed process' keepevicted=true"`
6. You recently ingested a custom application log file into Splunk. When searching the contents in Splunk web, multiple lines of the log files are incorrectly merged together in a single event. What is the most likely cause of this issue?
	* The `LINE_BREAKER` attribute is incorrectly set in **props.conf.**
7. How would you add color in a tabular output based on the values of the cells?
	* Format the table visualization **using the data overlay option with Heat map selected**
8. You have a list of raw logs from Splunk. Which regex command would you use to add IP addresses in the selected fields?
	*  `index =* | rex "^Id{1,3}\.\d{1,3}\.\d{1,3}\. \d(1,3}\s\`
9. Which tool enables you to create interactive dashboards that provide more information when data points are clicked?
	* Drilldown
10. You are normalizing data that is examining login events between two different sources. You see some discrepancies between the names of fields in your data and the names expected by the data models. What should you do to normalize these field names?
	* **Create field aliases**.
11. What is the difference between the eventCount property and the scanCount property?
	* **The eventCount property represents the number of events that search returns, whereas the scanCount property represents the total number of events scanned**
12. What is the difference between an events index and a metrics index?
	* **An events index can hold any type of data, whereas a metrics index can hold only metrics data.** 
13. You tagged a few IP addresses based on purpose, patch status, and location as follows: 12.24.18.15: main,patch, south 12.24.18.15: backup,new,west 12.24.18.15: main,new, east Which command gives you all the events that are patched or located in the east?
 * `index=main tag=patch OR tag=east`
14. Which configuration will create a token for a drop-down list for form input?

```
<input type="dropdown" token="event_dropdown">
	<label>Select an event type</label>
	<default›event1</default›
	<choice value="event1">event1</choice>
	<choice value="event2">event2</choice>
</input>
```
15.Which Search Processing Language (SPL) command orders the results into a data structure that you need to generate visualizations such as charts?

* stats

16.An analyst is running the following search command in Splunk, but it's taking a long time to load. How can the search be modified in order to
make it more efficient? `index=* sourcetype=WinEventLog EventCode=* sc ip=192.168.10.2`

```
index=Windows sourcetype=WinEventLogEventCode=*b src_ip=192.168.10.2
```

17.You log out of Splunk after running an ad-hoc search. After five minutes, you must export the results of the search. How would you achieve this?

* Log in to Splunk. **Because ad-hoc jobs are kept for 10 minutes, locate the job in the Jobs manager and download the results**


18.Log in to Splunk. Because ad-hoc jobs are kept for 10 minutes, locate the job in the Jobs manager and download the results

* `| eval KBYTES= (bytes/1000)`

19.You must analyze the trend of the 95th percentile response time split by the unique API endpoints your application serves. What is one way to
achieve this?

* Retrieve all relevant events and pipe them to `timechart p95(response_time) by api_endpoint`

20 How can you improve your search query performance?

* Narrow the timeframe as much as possible.

21 An Engineer creating a **dri11 down** for a table of values in Splunk. Upon clicking the data in the table, a separate URL should be opened and the
data is searched for using the following format: `https://searchenginercom/?q=<value>` Which configuration is correct to enable this drilldown?

```
<drilldown>
	<link>
		https://searchengine.com/?q=$click.values
	</link>
</drilldown>
```

22 You have a field in a root dataset named `primary_building` and it's set to required. You create two more datasets that inherit from your root
dataset, but you do not see primary building when setting filters to view only required fields. What must you do to make this field appear?

* **Change the primary_building field to required in the child datasets**

23 You start ingesting data from **access.log** and **error.log** from your web server. You configure a common source type web:log for the logs coming from the web server. When searching for the events in Splunk, **access.log** events are formatted correctly, but error.log events are not broken into proper events. For example, multiple log entries are grouped into one event. Why might that be?

* The **access.log and error.log** might have different formats and need to use different source types

24 How can self-describing data be used with the Splunk field extractor to ensure all relevant fields are extracted and used?

* Use the delimiter option to separate the fields from the data.

25 7. What does the following search command do? `eventtype="windows _login" makemv delim=";"` users

**The command separates multi-value fields.**

26 An application sends alerting-based data to Splunk. When an event is processed, Splunk hasn't enriched the dataset with relevant fields for this type of information. What Common Information Model (CIM) data model would improve this?

* **The alerts dataset**

27 When an analyst is using a transaction to correlate events, they're not able to see the information grouped by the time span that they need. How can the following search be modified in order to accomplish this?

U* se the **maxspan** argument in the transaction command

28 You must generate a report of top users logging in to your application. The field "user" in your events represent a unique user. How would generate this report using the fields sidebar?

* Use the prebuilt report "top values" from the fields.

29 11. Which Splunk Search Processing Language (SPL) command can you use to group conceptually-related events together within a 25 second window?

* **transaction**

30 You configured your Splunk application to extract HTTP status codes from a web service log as a field called http status. You are now creating aworkflow that will only appear when http status is between 400-500. This workflow action will send the information to an external data manager that will generate bug reports. What kind of workflow would you create to do this?

* **POST**

31 In a distributed Splunk Enterprise environment, where is the Splunk Common Information Model (CIM) add-on installed?

* **Search head**

32 An organization has two Splunk deployments, Cloud and Enterprise. An Engineer is attempting to run searches from the cloud Search Head to view data from both the Enterprise and Splunk Cloud instances, but it's not working. Why?

* **A Hybrid Search Head is needed for this task, and must be on-premise**.

33 An Architect is designing a Splunk Enterprise deployment and is working on a design that includes an indexer cluster. What must the Architect consider when deciding how to include redundancy for the peer nodes in case of failure?

* **The manager will coordinate the process of reproducing the failed peers buckets on the remaining nodes.**

34 You are working with an application with a lot of duplicate and obsolete tags. You all the unnecessary tags. Later, you see that a field in one report is no longer functioning. What should you do next?

* **Check if one of the deleted tags was previously referenced to an existing key-value pair.**

35 You are trying to plot both the average response time and the median response time against time in the same graph using the following Search
Processing Language (SPL) query:

```
index=main sourcetype=access combined wcookie
timechart avg (response time) as AvgReponseTime span=30m
stats median (AvgResponseTime)
```

* **You must use the SPL command `eventstats` instead of stats to retain the `AvgResponseTime`**.

36 An organization is attempting to reduce the impact should a manager node fail in a clustered environment. They want to add another manager node into the cluster for redundancy. How can they implement this?

* **The second manager can be a standby manager node in case of the primary node's failure**

37 You are using a search macro and you run into this error: Error in
'SearchParser': The search specifies a macro that
cannot be found Your spelling is correct and you have the necessary permissions. What could be the reason for this error?

* **The definition of the search macro is wrong and you did not supply the required arguments when using the search macro.**

