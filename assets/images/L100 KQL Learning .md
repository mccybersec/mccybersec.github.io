# L100 KQL Learning

This article is about an introduction to KQL - Kusto Query Language - a read-only language used to query data in many Microsoft services such as Azure Data Explorer (ADX), Azure Monitor Logs, Microsoft Sentinel and Azure Resource Graph. It is a basic article on the most used operators.
Quick links to the various sections:
- [summary](#summary)
- [pipe operator](#pipe)
- [take operator](#take)
- [distinct operator](#distinct)
- [where operator](#where)
- [project operator](#project)
- [project-away operator](#projectaway)
- [extend operator](#extend)
- [summarize operator](#summarize)
- [join operator](#join)
- [let command](#let)



## <a name="summary">summary</a>
In the Log Analytics Workspace, data is stored in tables - each with its own schema (column names and corresponding types).<br>
In the examples below we will use the SigninLogs table - created and populated with the Sentinel Entra Connector. This table contains information relating to the type of sign-in carried out, the entity involved, the final application, the agent used, whether the activity was successful or not, the IP from which it was carried out and many more information accessible [here](https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/signinlogs). <br>

The goal of a KQL query is to obtain information and insights from the stored data rows and to do this various operators are used to manipulate their display. Everything is always in the form of a table: we start from the source (table), apply filters on the data (obtaining a table), apply aggregations on the data (the result of which is a table), and apply operators to improve readability . Please remember that a visualization system is valid not when it shows data in a pleasant format but when it addresses the intentions and needs of users.


## <a name="pipe">_pipe_ operator</a>
A KQL query is based on the concept of a pipeline with the pipe operator represented by the ```|``` character: it simply retrieves the output of everything before the operator, making it input for the immediately following command.
<br>

## <a name="take">_take_ operator</a>
The ```take``` operator is very useful for retrieving random samples of data from a table. <br>
As mentioned previously, each table has its own schema: a ```take 10``` is very useful for immediately having information on the schema and also on the possible values that the various columns can have. <br>

```
SigninLogs
| take 10
```

<div align="center">
    <img src="../post9/take.png">
</div>


## <a name="distinct">_distinct_ operator</a>
The ```distinct``` operator is useful for understanding which different values are present for a given column<br>

```
SigninLogs
| distinct ResultType
```
<div align="center">
     <img src="../post9/distinct.png">
</div>

Note how it is possible to have multiple parameters for the distinct: the following query returns each distinct UserPrincipalName, IpAddress pair, useful for understanding for each UserPrincipalName from which IpAddress it has performed sign-ins.

```
SigninLogs
| distinct IPAddress, UserPrincipalName 
```

## <a name="where">_where_ operator</a>
The ```where``` operator is used to filter data based on the values stored in the columns - keeping those that satisfy a predicate. Columns can contain numbers - and therefore use predicates like [Less, Greater, Equals](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/numerical-operators) - strings - and then use predicates like [Equals, Contains, startswith, Regex](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/datatypes-string-operators) or datetime. You can use Logical and and Logical or.

```
SigninLogs
| where Location == "US"  
```
<div align="center">
    <img src="../post9/where.png">
</div>

<div align="center">
    <img src="../post9/whereand.png">
</div>

## <a name="project">_project_ operator</a>
If the number of rows is reduced with the ```where``` operator, the number of columns is reduced with the ```project``` operator - maintaining only those of interest.

```
SigninLogs
| project UserDisplayName, AppDisplayName, TimeGenerated
```
<div align="center">
    <img src="../post9/project.png">
</div>

Notice how with the ```project``` operator it is also possible to reorder the columns - the _TimeGenerated_ coloumn has been moved to the last value. you can also rename the columns with the following syntax

```
SigninLogs
| project User=UserDisplayName, App=AppDisplayName, Time=TimeGenerated
```

## <a name="project-away">_project-away_ operator</a>
If the ```project``` operator describes the columns you want to keep, the ```project-away``` operator mentions the ones you want to delete.

```
SigninLogs
| project-away UserDisplayName
```

## <a name="extend">_extend_ operator</a>
The ```extend``` operator is used to insert a new column to the temporary table that is being created with the query. <br>
Please remember that KQL is a read-only language: the raw data is not modified. <br>
The new column can be the concatenation of two column values, it can be the enrichment obtained with information in another table, it can depend on the value of other columns.

```
SigninLogs
| extend infolocatin = iff("UserDisplayName", true, false)
```
<div align="center">
    <img src="../post9/extend.png">
</div>

The previous query shows how to create a new boolean column _infolocationavailable_ - which has the value True if and only if the Location field is not null.

## <a name="summarize">_summarize_ operator</a>
The ```summarize``` operator is used as an aggregation function for data analysis such as calculating the maximum, minimum, average. <br>
It is always used together with other operators. <br>
Let's see some examples

```
SigninLogs
| summarize count() by Location
```
<div align="center">
    <img src="../post9/summarize1.png">
</div>

The query shows the number of logs in the _Signinlogs_ table for each distinct location value. <br>
Also in this case there can be more parameters for the operator.

```
SigninLogs
| summarize count() by Location, UserDisplayName
```

You may be interested in getting the latest sign-in event for each user. Below is the query for this purpose.

```
SigninLogs
| summarize arg_max(TimeGenerated, *) by UserDisplayName
```
<div align="center">
    <img src="../post9/summarize2.png">
</div>

What if you want to get the sign-in number hour by hour? And maybe create a timechart about it?<br>
In this case the ```bin``` operator is used.

```
SigninLogs
| summarize count() by bin(TimeGenerated, 1h)
| render timechart 
```

<div align="center">
    <img src="../post9/summarize3.png">
</div>

Simply the ```bin``` operator creates different time buckets based on the timerange passed as a parameter: each log is inserted into the corresponding bucket based on the value of _TimeGenerated_. At this point the count is carried out for each bucket. The result is a table with two columns the timerange and number of entries for each of it; the ```render``` operator is applied to graph it.

## <a name="join">_join_ operator</a>
To correlate data in different tables, the ```join``` operator is used. The idea is to carry out enrichment based on a shared key value. This value can be saved in columns with the same name or not - and in this case it is necessary to specify which property does the linking.

```
SigninLogs
| join SecondTable on $left.property == $right.property 
```

## <a name="let">_let_ operator</a>
You can declare volatile variables in KQL using the ```let``` operator. A variable can be a single value, a list, or a table. It can be initialized statically, from an external source or as a result of a previous query.

```
let location = "US";
SigninLogs
| where Location == location
```

<div align="center">
    <img src="../post9/let1.png">
</div>

```
let Locations = 
SigninLogs
| take 100
| distinct Location;
```


For more information don't hesitate to contact me!<br>
Thank you for taking time to read.

Stay tuned!<br>
_Mario_