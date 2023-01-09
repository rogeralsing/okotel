# Getting Started with Grafana

To use Grafana Tempo and Grafana Loki via docker compose, you need to first make sure that you have Docker installed on your local machine. Once you have Docker installed, you can create a docker-compose.yml file that defines the services for Grafana Tempo and Grafana Loki.

To run Grafana Tempo and Grafana Loki, you can use the following commands:


```
# Start the services defined in the docker-compose.yml file
docker-compose up
```

And to stop the services, you can use the following commands:

```
# Stop and remove the containers created by docker-compose
docker-compose down

# alternatively -v to remove any previously used disc volumes
#docker-compose down -v
```

Once the services are running, you can access the Grafana dashboard by visiting http://localhost:3000 in your browser. 

## Navigating Grafana

Once in Grafana, go to the "Explore" side-menu option.
From the explore view, you can then select Tempo or Loki as your datasource.

<img src="./explore.png">

### Tempo

To use Tempo, you can select the "Tempo" datasource from the dropdown menu.

<img src="./tempo.png">

### Loki

To use Loki, you can select the "Loki" datasource from the dropdown menu.

<img src="./loki.png">

Using LogQl to query Loki, you can use the following query to find all logs that contain the string "3139ba6b320f465e9e48a8195e7496b7" in the body field.

> Note: we are assuming the logs are in JSON format and not plain text here.

We assume the system is using NLog with JSON console scraping by the DataDog Agent.

In order to query JSON logs in Loki, you need to use a log pipeline to parse the log line as JSON and then use a log filter to filter the logs.

This can be done using the following query:

```
{service_name="CentralSystem"} | json | line_format "{{.body}}" |~ "3139ba6b320f465e9e48a8195e7496b7"
```

This looks like a lot, but it's quite simple.
Let's break down the query:

The first part `{service_name="CentralSystem"}` is a label filter. This will filter the logs to only include logs that have the label `service_name` with the value `CentralSystem`.

The second part `| json | line_format "{{.body}}"` is a log pipeline. This will parse the log line as JSON and then format the log line to only include the `body` field.

The third part `|~ "3139ba6b320f465e9e48a8195e7496b7"` is a log filter. This will filter the logs to only include logs that contain the string `3139ba6b320f465e9e48a8195e7496b7`, ignoring case.

Query operators:

* `|` is a pipe operator. It is used to chain multiple operators together.
* `!` is a negation pipe operator. It is used to negate the result of the next operator.
* `=` is case sensitive equality. 
* `~` is case insensitive equality. 
* ´json´ is a log pipeline operator. It is used to parse the log line as JSON.
* `line_format` is a log pipeline operator. It is used to format the JSON to plain text.
* `{{.body}}` is the format template. it allows you to combine text with fields from the JSON log line.

Loki has a concept of "Context" for log filtering.
Once you have applied a filter, and found the log line you are looking for. you can then open the context and see the log lines before and after the log line you are looking for.

You can do so by clicking the "Show Context" hover button on the log row:
<img src="./showcontext.png">

This will bring up the log context dialog

<img src="./lokicontext.png">

Where you can see the selected log row in the middle, and the logs before and after that entry.

