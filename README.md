# Splunk# 

In this project I explore a large dataset of over 500,000 events containing many different attacker TTPs using the powerful SIEM tool, [Splunk Enterprise](https://www.splunk.com/en_us/products/splunk-enterprise.html). The goals of this project is to: 

Use many of the available Search Processing Language (SPL) tools to craft efficient and complex searches: [Splunk and SPL](#splunk-and-spl)

## Splunk and SPL

In this section I will focus on the SIEM capabilities of Splunk and go through its many data analysis tools. I will also use Splunk Processing Language (SPL) to conduct various searches, filters, transformations, and visualizations. 

### Splunk as a SIEM 

To begin creating basic SPL commands, I will use a VM host setup with a Splunk Index named **main** containing Windows Security, Sysmon, and other logs. 

For some starter searches I first query the index for the term "UNKNOWN" using `index=main "UNKNOWN"`:

![image](https://github.com/user-attachments/assets/420301a9-0f56-4529-95fb-c6a51ee31fde)


Then I can modify that same query with wildcards to find all occurrences of "UNKNOWN" with any amount of characters before and after it:

![image](https://github.com/user-attachments/assets/df05d3bc-50ce-4093-b067-84a45d74406a)


The wildcards return more results as the search criteria becomes less strict. 

Splunk automatically identifies data fields from the events like source, souretype, host, and EventCode. For example, from the previous search I can see some of the hosts that were found: 

![image](https://github.com/user-attachments/assets/b1de07ad-3cf5-463f-b586-c92c679ededa)


Then I can create queries using these data fields combined with comparison operators to filter based on the values found for each data field. With this information, I do a search for all records where the host is "waldo-virtual-machine" using `index="main" host="waldo-virtual-machine"`:

![image](https://github.com/user-attachments/assets/2f3e2530-324d-4435-8236-4a7c83deeb4b)

Using a pipe "|" I can direct the output of a search into a command, similar to Linux. For the data fields, SPL offers a **fields** command that I can use to remove and add specific fields from the results. 

With the fields command I conduct a search on all Sysmon events with EventCode 1 but remove the "User" field from the results. This will output all the usual results except the ones where the user initiated the process:

![image](https://github.com/user-attachments/assets/7edcf377-c362-4ca9-8dfb-3595afacbf31)


Another useful command is **table** which can be used to change the display of the results into a table with the desired columns. 

With the Sysmon EventCode 1 results I can create a table that only displays the time, host, and image fields:

![image](https://github.com/user-attachments/assets/fbe845e3-5292-4c11-801a-fb7e5ea75cc4)


If I wanted to use a different name for a field then I can use the command **rename** to change it in the results. For example, I can change the _time field to be "time and date":


Another handy command is **dedup** which deletes all duplicate events based on a specified field:

For example image field will have many duplicates, but with dedup many of them are filtered out.


With the **sort** command I can sort the results in ascending or descending order based on a specified field. Here I sort the results by the time they occurred and in descending order so I can see the most recent results:

![image](https://github.com/user-attachments/assets/63eaf7fd-b269-4eab-bc97-8d456474922f)


The **stats** command allows me to compute statistical operations on the results to better organize/visualize it. Using the **count** operation I compile the results to show the number of times that each Image created an event at a certain time:

![image](https://github.com/user-attachments/assets/4fa2faec-f36f-4354-bc44-0e1fe04388a0)


To further expand on the data visualization aspect of SPL, there is the **chart** command that is very similar to **stats** but outputs the results into a table-like data visualization.

I can create a chart with the previous example of taking the count of events that and Image created at a specific time:

![image](https://github.com/user-attachments/assets/5947b1e6-7285-448d-9113-686e4c86a9ea)

I use SPL's capability to do subsearches to filter out large sets of data. I start by creating a very simple search to get all Sysmon events with EventCode 1. 

Using the logical NOT keyword I can filter out all the results of a subsearch from the results of this main search:

```
NOT
	[ search index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
	| top limit=100 Image
	| fields Image ]
```

The subsearch will find all Sysmon events with Event Code 1 and returns only the 100 most common values of the Image field. Therefore, all of these results will be removed from the main search:

![image](https://github.com/user-attachments/assets/ab8a6a64-ba82-4914-b8cd-7896ceb9d2b4)


Filtering out these events will result in a search that provides a look into some of the Event Code 1 events that feature more rare and unique instances of programs being used. 

