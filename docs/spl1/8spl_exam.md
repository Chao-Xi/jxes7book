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
	* Create field aliases.
11. What is the difference between the eventCount property and the scanCount property?
	* The eventCount property represents the number of events that search returns, whereas the scanCount property represents the total number
of events scanned
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
index=Windows sourcetype=WinEventLogEventCode=*
src_ip=192.168.10.2
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



