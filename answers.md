Your answers to the questions go here.
Collecting Metrics:
Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.
# Set the host's tags (optional)
tags:
   - mytag
   - env:prod
   - role:database
   - test:polflip
   
   [screenshot] (https://www.dropbox.com/s/ibxtsvhmy78ks83/tags.PNG?dl=0)

Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.
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

  

Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

/etc/datadog-agent/checks.d
import random
# the following try/except block will make the custom check compatible with any Agent version
try:
    # first, try to import the base class from old versions of the Agent...
    from checks import AgentCheck
except ImportError:
    # ...if the above failed, the check is running in Agent version 6 or later
    from datadog_checks.checks import AgentCheck

# content of the special variable __version__ will be shown in the Agent status page
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
 [Screenshot](https://www.dropbox.com/s/ydkw4al6jq2mhdw/changeInterval.PNG?dl=0)
 
 As a result, the metric appears on Metric Explorer:
 [Screenshot](https://www.dropbox.com/s/hgr0f9al00q2gkz/my_metricUI.PNG?dl=0)

Bonus Question Can you change the collection interval without modifying the Python check file you created?
Yes, this goes in the yaml file



------
Visualizing Data:
Utilize the Datadog API to create a Timeboard that contains:

Your custom metric scoped over your host.
I have use curl to post the metric:
And I have created an application key as well on the UI
procafort_dashboard.json:
from datadog import initialize, api

options = {
    'api_key': '6d9ffcce9cd2c6fb20ef4f0085356d1f',
    'app_key': '<04d117a9f3b1e1728cdf7738a20bc3062f2ff7d6>'
}

initialize(**options)

title = 'my_dashboard'
widgets = [{
    'definition': {
        'type': 'timeseries',
        'requests': [
            {'q': 'avg:my_metric{*}'}
        ],
        'title': 'My_Metric'
    }
}]
layout_type = 'ordered'
description = 'A dashboard with custom metric.'
is_read_only = True
notify_list = ['user@domain.com']

A nd then POST it:


Any metric from the Integration on your Database with the anomaly function applied.
postgreql.rows_deleted

Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket
 {"definition": {
      "type": "timeseries",
      "requests": [
        {"q": "avg:mymetric{*}.rollup(sum, 3600)"}
      ],
      "title": "Sum rollup of mymetric values recorded in the last hour" 
Please be sure, when submitting your hiring challenge, to include the script that you've used to create this Timeboard.

Once this is created, access the Dashboard from your Dashboard List in the UI:

Set the Timeboard's timeframe to the past 5 minutes
Take a snapshot of this graph and use the @ notation to send it to yourself.
Bonus Question: What is the Anomaly graph displaying?
-------------
