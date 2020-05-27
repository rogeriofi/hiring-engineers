## Your answers to the questions go here.

##### Collecting Metrics:
##### Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

*A: Tag has been added to /etc/datadog-agent/datadog.yaml:
hostname: ECOMMERCE

<img src="https://github.com/rogeriofi/datadog_test/blob/master/tag.png" width="800" height="403">

##### Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

*A: MySQL DB has been installed and the integration added from the WebUI. After it, it is a simple case to start the collection configuring the yaml file with the DB connection over: 
/etc/datadog-agent/conf.d/mysql.d/conf.d

##### Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

*A: In /etc/data-agent/checks.d I created the custom check:
```
[root@konakart checks.d]# cat custom_check.py 
from datadog_checks.checks import AgentCheck
from random import randint
class Check(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', randint(0,1000))
[root@konakart checks.d]#
```

##### Change your check's collection interval so that it only submits the metric once every 45 seconds.

*A:
Over the path :
[root@konakart conf.d]# cat custom_check.yaml 
init_config:
instances:
  - min_collection_interval: 45
  
##### Bonus Question Can you change the collection interval without modifying the Python check file you created?

*A: Simply change the config /etc/datadog-agent/conf.d/custom_check.yaml as it shows in the precious answer.

##### Visualizing Data:
##### Utilize the Datadog API to create a Timeboard that contains:

##### Your custom metric scoped over your host.
##### Any metric from the Integration on your Database with the anomaly function applied.
##### Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket
##### Please be sure, when submitting your hiring challenge, to include the script that you've used to create this Timeboard.

*A Created a script to create the dashboard via API:

```
[root@konakart ~]# cat datadog_script.py 
from datadog import initialize, api
options = {
    'api_key': 'e5abbe9bfb4400015c25d3dc0332fbf8',
    'app_key': 'd3ae82b5beec8ae71522fb81b435d020cfcf6f1f',
    'api_host': 'https://api.datadoghq.eu'
}
initialize(**options)
title = 'my_metric dashboard Test'
description = 'Testing adding a new dashboard for my_metric using datadog API.'
layout_type = 'ordered'
widgets = [{
    'definition': {
        'type': 'timeseries',
        'requests': [
            {'q': 'avg:my_metric{host:ECOMMERCE}'}
        ],
        'title': 'my_metric on ECOMMERCE'
    }
},
    {
    'definition': {
        'type': 'timeseries',
        'requests': [
            {'q': "anomalies(avg:mysql.performance.user_time{host:ECOMMERCE}, 'basic', 3)"}
        ],
        'title': 'Mysql CPU time per user'
    }
},
    {
    'definition': {
        'type': 'query_value',
        'requests': [
            {'q': 'my_metric{host:ECOMMERCE}.rollup(sum, 3600)'}
        ],
        'title': 'my_metric rollup function'
    }
}]
api.Dashboard.create(title=title,widgets=widgets,description=description,layout_type=layout_type)
```


##### Once this is created, access the Dashboard from your Dashboard List in the UI:

##### Set the Timeboard's timeframe to the past 5 minutes
##### Take a snapshot of this graph and use the @ notation to send it to yourself.

I couldn't change to the last 5 minutes.

<img src="https://github.com/rogeriofi/datadog_test/blob/master/my_metric.png" width="800" height="403">


##### Bonus Question: What is the Anomaly graph displaying?

*A The check itself is checking the CPU time used by MySQL. The gray area represented in the dashboard is the baseline taken from historical data. If anything deviates from the baseline a red line is diplayed to alert of abnormal usage. THe meaning is that the DB could be faulty or not performing queries in a tinmely fashion.


##### Monitoring Data
##### Since you’ve already caught your test metric going above 800 once, you don’t want to have to continually watch this dashboard to be alerted when it goes above 800 again. So let’s make life easier by creating a monitor.

##### Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

##### Warning threshold of 500
##### Alerting threshold of 800
##### And also ensure that it will notify you if there is No Data for this query over the past 10m.
##### Please configure the monitor’s message so that it will:
##### Send you an email whenever the monitor triggers.
##### Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.
##### Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.
<img src="https://github.com/rogeriofi/datadog_test/blob/master/alert1.png" width="800" height="403">
<img src="https://github.com/rogeriofi/datadog_test/blob/master/alert2.png" width="800" height="403">
##### When this monitor sends you an email notification, take a screenshot of the email that it sends you.
<img src="https://github.com/rogeriofi/datadog_test/blob/master/email_alert.png" width="800" height="403">

##### Bonus Question: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:

##### One that silences it from 7pm to 9am daily on M-F,
<img src="https://github.com/rogeriofi/datadog_test/blob/master/downtime1.png" width="800" height="403">
##### And one that silences it all day on Sat-Sun.
<img src="https://github.com/rogeriofi/datadog_test/blob/master/downtime2.png" width="800" height="403">
##### Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification.
<img src="https://github.com/rogeriofi/datadog_test/blob/master/notify.png" width="800" height="403">



##### Collecting APM Data:

Bonus Question: What is the difference between a Service and a Resource?

*A I will be honest here and put my understanding instead of reading Datadog material. A Resource means a server, an application, DB, etc, it will be standalone checks that when displayed in the monitoring will be your resource.
On the other hand a service can be a number of resources combined, creating an actual Business service. For e.g. You have an application server and a DB server as resources being monitored. The service ECOMMERCE are the 2 combined.
For some reason in the datadog dashboards that understanding seems backwards.

##### Provide a link and a screenshot of a Dashboard with both APM and Infrastructure Metrics.

*A I did not follow the creation for the application given. I instrumented my own to see better data.
Here is a snap from my APM map view:
<img src="https://github.com/rogeriofi/datadog_test/blob/master/apm.png" width="800" height="403">

some metrics from the APP (Tomcat as well):

<img src="https://github.com/rogeriofi/datadog_test/blob/master/app_metrics.png" width="800" height="403">



##### Please include your fully instrumented app in your submission, as well.

##### Final Question:
##### Datadog has been used in a lot of creative ways in the past. We’ve written some blog posts about using Datadog to monitor the NYC Subway System, Pokemon Go, and even office restroom availability!

##### Is there anything creative you would use Datadog for?
*A There could be a monitoring of supermarket parking spaces displayed in a website. TO make sure you don't go when you know there is going to be a queue.
