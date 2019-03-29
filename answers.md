Your answers to the questions go here.
#Collecting Metrics:

#Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.

Set the host's tags (optional)
tags:
   - mytag
   - env:prod
   - role:database
   - test:polflip
   
   ![Screenshot](https://www.dropbox.com/s/ibxtsvhmy78ks83/tags.PNG?dl=0)

#Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.
I installed directly with


vagrant@ubuntu-xenial:~$ sudo apt-get install postgresql
Reading package lists... Done
Building dependency tree
Reading state information... Done
Suggested packages:
  postgresql-doc
The following NEW packages will be installed:
  postgresql
0 upgraded, 1 newly installed, 0 to remove and 5 not upgraded.
Need to get 0 B/5,450 B of archives.
After this operation, 59.4 kB of additional disk space will be used.
Selecting previously unselected package postgresql.
(Reading database ... 69899 files and directories currently installed.)
Preparing to unpack .../postgresql_9.5+173ubuntu0.2_all.deb ...
Unpacking postgresql (9.5+173ubuntu0.2) ...
Setting up postgresql (9.5+173ubuntu0.2) ...


vagrant@ubuntu-xenial:~$ sudo passwd postgres
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
vagrant@ubuntu-xenial:~$ sudo service postgresql start
vagrant@ubuntu-xenial:~$ su postgres
Password:

postgres@ubuntu-xenial:/home/vagrant$ createdb datadog_db
postgres@ubuntu-xenial:/home/vagrant$ psql -d datadog_db
create user datadog with password 'datadog';
grant SELECT ON pg_stat_database to datadog;

datadog_db-# psql -h localhost -U datadog postgres -c "select * from pg_stat_database LIMIT(1);" && echo -e "\e[0;32mPo
stgres connection - OK\e[0m" || echo -e "\e[0;31mCannot connect to Postgres\e[0m"

vagrant@ubuntu-xenial:/etc/datadog-agent/conf.d/postgres.d$ sudo vi conf.yaml
 
 init_config:

instances:
  - host: localhost
    port: 5432
    username: datadog
    password: datadog
    
    [Screenshot](https://www.dropbox.com/s/lvz5mtjfxsacui6/dbintegration.PNG?dl=0)
    
  After configuring the integration, we can see it running on 
  https://app.datadoghq.com/account/settings#integrations/postgres
  
  It appears on the integration
  [Screenshot](https://www.dropbox.com/s/9a2e2fswqephid8/dbintegration2.PNG?dl=0)
  
  And running the datadog-agent status command
      postgres (2.5.0)
    ----------------
      Instance ID: postgres:6ded5fb94d97f938 [OK]
      Total Runs: 27
      Metric Samples: Last Run: 45, Total: 1,215
      Events: Last Run: 0, Total: 0
      Service Checks: Last Run: 1, Total: 27
      Average Execution Time : 29ms

  

#Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000

/etc/datadog-agent/checks.d
import random
 the following try/except block will make the custom check compatible with any Agent version
try:
    # first, try to import the base class from old versions of the Agent...
    from checks import AgentCheck
except ImportError:
    # ...if the above failed, the check is running in Agent version 6 or later
    from datadog_checks.checks import AgentCheck

 #content of the special variable __version__ will be shown in the Agent status page
__version__ = "1.0.0"


class RandomCheck(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', random.randint(0,1000))
 [Screenshot](https://www.dropbox.com/s/4nlqgxty4ug4frf/my_metric.PNG?dl=0)   

:/etc/datadog-agent/conf.d/my_check.yaml
init_config:
instances:
    [{}]
    
    

Change your check's collection interval so that it only submits the metric once every 45 seconds.
init_config:
instances:
 - min_collection_interval: 45
 ![Screenshot](https://www.dropbox.com/s/ydkw4al6jq2mhdw/changeInterval.PNG?dl=0)
 
 As a result, the metric appears on Metric Explorer:
 ![Screenshot](https://www.dropbox.com/s/hgr0f9al00q2gkz/my_metricUI.PNG?dl=0)

Bonus Question Can you change the collection interval without modifying the Python check file you created?
Yes, this goes in the yaml file



------
#Visualizing Data:
#Utilize the Datadog API to create a Timeboard that contains:

Your custom metric scoped over your host.
I have use Postman defining the api_key and the app_key there to post the metric:
And I have created an application key as well on the UI
procafort_dashboard.json:

{
  "title" : "My_Excercise",
  "widgets" : [
    {"definition": {
      "type": "timeseries",
      "requests": [
        {"q": "avg:my_metric{host:ubuntu-xenial}"}
      ],
      "title": "my_metric"
    }},

Any metric from the Integration on your Database with the anomaly function applied.
 {"definition": {
   "type": "timeseries",
   "requests": [
     {"q": "anomalies(avg:postgresql.percent_usage_connections{*}, 'basic', 1)"}
   ],
   "title": "Anomaly graph for PostgreSQL connections"

Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket
  {"definition": {
   "type": "timeseries",
   "requests": [
     {"q": "avg:my_metric{*}.rollup(sum, 3600)"}
   ],
   "title": "Sum rollup of mymetric values recorded in the last hour"
 }}
 
Please be sure, when submitting your hiring challenge, to include the script that you've used to create this Timeboard.
Used Postman to run a POST POST "https://api.datadoghq.com/api/v1/dashboard?app_key=04d117a9f3b1e1728cdf7738a20bc3062f2ff7d6&api_key=49514af82afd9cde0bd302ba37201f49"
![Screenshot](https://www.dropbox.com/s/6cyt2byvj0z3yng/dashboard.PNG?dl=0)
Once this is created, access the Dashboard from your Dashboard List in the UI:

Set the Timeboard's timeframe to the past 5 minutes

Take a snapshot of this graph and use the @ notation to send it to yourself.

![Screenshot](https://www.dropbox.com/s/q044p8q3k3ecml4/notify.PNG?dl=0)

[Screenshot](https://www.dropbox.com/s/npje1oxyhkrkc5h/dashboardPostman.PNG?dl=0)

Bonus Question: What is the Anomaly graph displaying?
It is displaying the normal behaviour and it adapts over time. The more time is running, the more the algorith learns and can better identify anomalies
------------------------------------------------------------------------------------------------------------------------------
Monitoring Data
Since you’ve already caught your test metric going above 800 once, you don’t want to have to continually watch this dashboard to be alerted when it goes above 800 again. So let’s make life easier by creating a monitor.

Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:

Warning threshold of 500
Alerting threshold of 800
And also ensure that it will notify you if there is No Data for this query over the past 10m.
Please configure the monitor’s message so that it will:

Send you an email whenever the monitor triggers.

Create different messages based on whether the monitor is in an Alert, Warning, or No Data state.

Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.

When this monitor sends you an email notification, take a screenshot of the email that it sends you.

Solution:
Call: https://api.datadoghq.com/api/v1/monitor?application_key=04d117a9f3b1e1728cdf7738a20bc3062f2ff7d6&api_key=49514af82afd9cde0bd302ba37201f49
Body:
{
	"name": "My_Metric monitor",
	"type": "metric alert",
	"query": "avg(last_5m):avg:my_metric{host:ubuntu-xenial} > 800",
	"message": " @PROCAFORT.AT@GMAIL.COM\n\n{{#is_alert}} My_Metric on {{host.name}} with IP {{host.ip}} is at {{value}},[ALERT] Test custom for alert. {{/is_alert}}\n{{#is_warning}}[TEST] Warning My_Metric is above 500. {{/is_warning}}\n{{#is_no_data}} [TEST] No Data. {{/is_no_data}}",
	"tags": [
		"my_metric.monitor"
	],
	"options": {
		"notify_audit": false,
		"locked": false,
		"timeout_h": 0,
		"new_host_delay": 300,
		"require_full_window": false,
		"notify_no_data": true,
		"renotify_interval": "0",
		"escalation_message": "",
		"no_data_timeframe": 10,
		"include_tags": true,
		"thresholds": {
			"critical": 800,
			"warning": 500
		}
	}
}

[Screenshot](https://www.dropbox.com/s/8odu73dbkqvg0lz/monitor.PNG?dl=0)
[Screenshot] (https://www.dropbox.com/s/vl1pfwvrxgfof3d/monitorEmail.PNG?dl=0)
[Screenshot](https://www.dropbox.com/s/rfourlbhlmvjpt2/monitorUI.PNG?dl=0)



Bonus Question: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor:


One that silences it from 7pm to 9am daily on M-F,
Call:https://api.datadoghq.com/api/v1/downtime?application_key=04d117a9f3b1e1728cdf7738a20bc3062f2ff7d6&api_key=49514af82afd9cde0bd302ba37201f49
JSON Body:{
  "scope": "*",
  "monitor_tags": ["my_metric.monitor"],
  "message": "My_Metric Monitor notifications silenced Mo-Fri 7pm to 9am",
  "timezone": "CET",
  "start": 1555347600,
  "end": 1555398000,
  "recurrence": {
    "type": "weeks",
    "period": 1,
    "week_days": ["Mon", "Tue", "Wed", "Thu", "Fri"]     
  }
}
[Screenshot](https://www.dropbox.com/s/uyyq9gc7cwze1uh/downtime.PNG?dl=0)

And one that silences it all day on Sat-Sun.
[Screenshot](https://www.dropbox.com/s/8kz7nfh0ohrpqyc/downtimeweekend.PNG?dl=0)

Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification.
[Screenshot](https://www.dropbox.com/s/8hs24ng830yj0tm/downtimeNotification.PNG?dl=0)


-------------
