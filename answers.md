Your answers to the questions go here.
Collecting Metrics:
Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog.
# Set the host's tags (optional)
tags:
   - mytag
   - env:prod
   - role:database

Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.
I installed directly with
vagrant@ubuntu-xenial:~$ sudo apt-get install postgresql
And then, create a role 
vagrant@ubuntu-xenial:~$   sudo su postgres -c "psql -c \"CREATE ROLE datadog SUPERUSER LOGIN PASSWORD 'datadog'\" "
and a database instance
# Create WTM database
  sudo su postgres -c "createdb -E UTF8 -T template0 --locale=en_US.utf8 -O datadog wtm"
  
  


Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.
Change your check's collection interval so that it only submits the metric once every 45 seconds.
Bonus Question Can you change the collection interval without modifying the Python check file you created?
