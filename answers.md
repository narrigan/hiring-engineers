# Prerequisites - Setup the environment

For this exercise I chose to spin up a blank ubuntu EC2 instance on AWS. For simplicity, I did not install the AWS integration as the intention of this assessment was to work with the agents, not the AWS integration.

## Installing the DataDog agent

Once the VM was provisioned I installed the DataDog agent on the Ubuntu box using the below command.

```DD_API_KEY=5b2274a00a64ceb64e64c289cf54f907 bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"```

# Collecting Metrics
### Add tags in the Agent config file and show us a screenshot of your host and its tags on the Host Map page in Datadog
Added tags to /etc/datadog-agent/datadog.yaml 
<img src="https://farm8.staticflickr.com/7805/46610412914_a290ebc8cf_b.jpg" alt="Tags in the datadog.yaml"></a>

Below screenshot from the ‘Host Map’ page showing the added tags in DataDog.
<img src="https://farm8.staticflickr.com/7891/40368874823_f671749f9f_b.jpg" alt="Tags in the datadog.yaml"></a>

### Install a database on your machine (MongoDB, MySQL, or PostgreSQL) and then install the respective Datadog integration for that database.

For the assessment I chose to install MySQL on the server 

<img src="https://farm8.staticflickr.com/7888/47333853221_918209b93a_b.jpg" alt="Installing MySQL"></a>

Created a database called ‘ddog’ 

<img src="https://farm8.staticflickr.com/7914/33458015878_575264cc30_o.png" alt="New ddgo DB"></a>

Created a table called customers and inserted some sample data

<img src="https://farm8.staticflickr.com/7895/33458015708_952e35d556_b.jpg" alt="New Customer Table "></a>

Created DataDog user

<img src="https://farm8.staticflickr.com/7899/33458015778_23bb422415_b.jpg" alt="DDog USer "></a>

Tested the New DataDog User has access to MySQL

<img src="https://farm8.staticflickr.com/7876/47333865501_24630ee661_b.jpg" alt="DDog User Access "></a>

Granted the datadag MySQL user access required for metrics 

<img src="https://farm8.staticflickr.com/7818/33458015658_501a4d4dd1_b.jpg" alt="DDog User Grants "></a>

Added /etc/datadog-agent/conf.d/mysql.d/mysql.yaml as per below

<img src="https://farm8.staticflickr.com/7882/40368633543_9e68d72126_o.png" alt="DDog User Grants "></a>

Installed the MySQL integration via the DataDog GUI

<img src="https://farm8.staticflickr.com/7926/47333881801_ae72721212_o.png" alt="MySQL GUI Installation"></a>

Restarted the DataDog-Agent and validated that the mysql check has passed

<img src="https://farm8.staticflickr.com/7880/32391854607_5f250cebe5_b.jpg" alt="Agent Restart"></a>

MySQL data is now displaying in DataDog

<img src="https://farm8.staticflickr.com/7898/47333881761_3f1eb1590d_b.jpg" alt="MySQL Dashboard"></a>

### Create a custom Agent check that submits a metric named my_metric with a random value between 0 and 1000.

I created a file the following 2 files:

/etc/datadog-agent/checks.d/randmetric.py 

/etc/datadog-agent/conf.d/randmetric.d/randmetric.yaml 

**randmetric.py**
```python
# the following try/except block will make the custom check compatible with any Agent version
try:
    # first, try to import the base class from old versions of the Agent...
    from checks import AgentCheck
except ImportError:
    # ...if the above failed, the check is running in Agent version 6 or later
    from datadog_checks.checks import AgentCheck

# content of the special variable __version__ will be shown in the Agent status page
__version__ = "1.0.0"

import random
class RandCheck(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', random.randint(0,1000))
```



I then validated that the custom agent check was responding correctly

<img src="https://farm8.staticflickr.com/7888/46610836314_819a986d49_b.jpg" alt="Validate Custom Check is working"></a>


I then restarted the agent using the following command
```sudo systemctl restart datadog-agent```
Looking at the agent status and the log tail I could see that the randmetric check was running successfully.

<img src="https://farm8.staticflickr.com/7838/32392184117_50969b5d80_b.jpg" alt="randmetric check was running successfully"></a>

### Change your check's collection interval so that it only submits the metric once every 45 seconds 
### Bonus Question Can you change the collection interval without modifying the Python check file you created

I updated the randmetric.yaml file to include the min_collection_interval: 45 as per below and restarted the datadog-agent and verified that the data was now being POSTED every 45 seconds by analysing the agent.log

**randmetric.yaml**
```yaml
init_config:

instances:
    - min_collection_interval: 45
 ```   
    
# Visualizing Data

### Utilize the Datadog API to create a Timeboard that contains: 
  ### Your custom metric scoped over your host. 
  ### Any metric from the Integration on your Database with the anomaly function applied.
  ### Your custom metric with the rollup function applied to sum up all the points for the past hour into one bucket

For simplicity I used cURL to POST the JSON payload containing the dashboard creation details. The JSON and the cURL request are below. The question was a little ambiguous whether to show the different metrics on one wiget or multiple widgets. I chose the former with the log scaling enabled to help view the MySQL CPU time metric which is on a smaller scale.


***create-dashboard.curl**
``` 
curl -X POST -H "Content-type: application/json" -d @dashboardpayload.json https://api.datadoghq.com/api/v1/dashboard?api_key=5b2274a00a64ceb64e64c289cf54f907&application_key=58598e9b28abb76440b275cfec142a9cd420d8ff
```

***dashboardpayload.json***

```json
{
    "title": "Niall's SE Dashboard",
    "widgets": [
        {
            "definition": {
                "requests": [
                    {
                        "q": "anomalies(avg:mysql.performance.cpu_time{host:i-0697fbf09f4a9c384}, 'basic', 2)",
                        "style": {
                            "line_width": "normal",
                            "palette": "orange",
                            "line_type": "solid"
                        },
                        "display_type": "line"
                    },
                    {
                        "q": "sum:my_metric{host:i-0697fbf09f4a9c384}.rollup(sum, 3600)",
                        "style": {
                            "line_width": "normal",
                            "palette": "dog_classic",
                            "line_type": "solid"
                        },
                        "display_type": "line"
                    },
                    {
                        "q": "avg:my_metric{host:i-0697fbf09f4a9c384}",
                        "style": {
                            "line_width": "normal",
                            "palette": "dog_classic",
                            "line_type": "solid"
                        },
                        "display_type": "line"
                    }
                ],
                "yaxis": {
                    "include_zero": false,
                    "scale": "log"
                },
                "type": "timeseries",
                "title": "mysql.performance.cpu_time with anomolies graphed against my_metric & my_metric rounded up to 1 hr bucket"
            }
        }
    ],
    "layout_type": "ordered"
}
```
### Once this is created, access the Dashboard from your Dashboard List in the UI: Set the Timeboard's timeframe to the past 5 minutes
URL to created dashboard
https://app.datadoghq.com/dashboard/hf6-6r6-hd3/nialls-se-dashboard?tile_size=m&page=0&is_auto=false&from_ts=1552085280000&to_ts=1552099680000&live=true

Last 4hrs (better view of the data)

<img src="https://farm8.staticflickr.com/7825/33458386898_4f9500830c_b.jpg" alt="Last 4hrs dashboard"></a>

### Take a snapshot of this graph and use the @ notation to send it to yourself.

Below is the email showing the comment and snapshot from the ‘Last 5mins @niall.arrigan@gmail.com’

<img src="https://farm8.staticflickr.com/7849/32392258777_1ab7068ea0_b.jpg" alt="Snapshot of last 5 mins"></a>

### Bonus Question: What is the Anomaly graph displaying?
    
I am using the basic anomaly algorithm which highlights values outside of a rolling range of expected values (shown by grey band). It adjusts quickly to changing conditions but has no knowledge of seasonality or long-term trends. I am using the value 2 which caters twice the deviation from the average before it considers it to be an anomaly. 

<img src="https://farm8.staticflickr.com/7896/46610956434_ee6f0c07f9_b.jpg" alt="anomaly"></a>

# Monitoring Data
### Create a new Metric Monitor that watches the average of your custom metric (my_metric) and will alert if it’s above the following values over the past 5 minutes:
### Warning threshold of 500
### Alerting threshold of 800
### And also ensure that it will notify you if there is No Data for this query over the past 10m.

Through the DataDog GUI I added a monitor as per below via ‘Monitors > Manage Monitors’

<img src="https://farm8.staticflickr.com/7865/40369108893_48f0df26fe_b.jpg" alt="Monitor"></a>

### Please configure the monitor’s message so that it will: Send you an email whenever the monitor triggers. Create different messages based on whether the monitor is in an Alert, Warning, or No Data state. Include the metric value that caused the monitor to trigger and host ip when the Monitor triggers an Alert state.

Notification to be sent to niall.arrigan@gmail.com with email text controlled by markup as per below.

<img src="https://farm8.staticflickr.com/7866/46611014234_d6d22e8a4e_b.jpg" alt="Monitor"></a>

### When this monitor sends you an email notification, take a screenshot of the email that it sends you.

Warning Email Notification:

<img src="https://farm8.staticflickr.com/7822/47334366621_994066c05a_b.jpg" alt="Warn Email"></a>

Recovery Email Notification

<img src="https://farm8.staticflickr.com/7812/47281773072_38f5ae76d7_b.jpg" alt="Recovery Email "></a>

### Bonus Question: Since this monitor is going to alert pretty often, you don’t want to be alerted when you are out of the office. Set up two scheduled downtimes for this monitor: 
### One that silences it from 7pm to 9am daily on M-F

<img src="https://farm8.staticflickr.com/7888/40369161743_3cecd3765b_b.jpg" alt="Monday-Friday Downtime "></a>

### And one that silences it all day on Sat-Sun.

<img src="https://farm8.staticflickr.com/7917/46419528175_b9a0dc9d89_b.jpg" alt="Weekend Downtime "></a>

### Make sure that your email is notified when you schedule the downtime and take a screenshot of that notification.

Weeknight downtime email

<img src="https://farm8.staticflickr.com/7875/46419551655_3963416360_o.png" alt="Monday-Friday Downtime Email"></a>

Weekend downtime email

<img src="https://farm8.staticflickr.com/7895/47334412371_cfbeb2afbb_o.png" alt="Weekend Downtime Email "></a>

# Collecting APM Data

I updated the datadog.yaml  to enable APM trace in the agent and configuring the environment to be the sandpit 

```yaml
# Trace Agent Specific Settings
#
apm_config:
  enabled: true
#   The environment tag that Traces should be tagged with
#   Will inherit from "env" tag if "none" is applied here
  env: sandpit
```

Tracing the Python app
install the Datadog Tracing library, ddtrace, using pip:

Then to instrument your Python application use the included ddtrace-run command.
<img src="https://farm8.staticflickr.com/7867/32392419087_57d4a6cc23_o.png" alt="ddtrace-run command"></a>

### Please include your fully instrumented app in your submission, as well.

**ddog_flask.py**
```python
from flask import Flask
import logging
import sys

# Have flask use stdout as the logger
main_logger = logging.getLogger()
main_logger.setLevel(logging.DEBUG)
c = logging.StreamHandler(sys.stdout)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
c.setFormatter(formatter)
main_logger.addHandler(c)

app = Flask(__name__)

@app.route('/')
def api_entry():
    return 'Entrypoint to the Application'

@app.route('/api/apm')
def apm_endpoint():
    return 'Getting APM Started'

@app.route('/api/trace')
def trace_endpoint():
    return 'Posting Traces'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port='8050')

```

### Bonus Question: What is the difference between a Service and a Resource?

A **service** is set of processes that do the same job. Examples include but are not limited to a webapp service, a database service, a storage service (like s3 bucket) or a discreet business logic service as part of a microservices architecture (shipping calculation service).
A **resource** is a particular action for a given service (typically an individual endpoint or query). E.g. ‘/’ would be the homepage resource for a webapp and ‘/api/’ could be another. For a SQL database: a resource is the query itself, such as SELECT * FROM users WHERE id = ?.

### Provide a link and a screenshot of a Dashboard with both APM and Infrastructure Metrics.

https://app.datadoghq.com/apm/service/flask/flask.request?end=1552173113602&env=sandpit&paused=true&start=1552172046935

<img src="https://farm8.staticflickr.com/7835/46419591915_e231f6202b_b.jpg" alt="APM Dashboard"></a>


# Final Question:
### Datadog has been used in a lot of creative ways in the past. We’ve written some blog posts about using Datadog to monitor the NYC Subway System, Pokemon Go, and even office restroom availability! Is there anything creative you would use Datadog for?

**Agtech in Australia:**
Australian agriculture is heavily impacted by the extremes in weather that can occur, e.g. droughts, heat waves. Using Internet of Things (IoT) devices to measure a farms key indicators (temperature, soil PH, rainfall, livestock feed levels) and integrating DataDogs agents into the IoT devices to monitor and alert when thresholds are reached. For example, automatically ordering more feed for livestock, or warning during periods of drought.  

**HomeBrew Intrusion Detection System (IDS)**
Leveraging DataDogs anomaly detection it could be possible to build a simple custom Intrusion Detection System (IDS) to determine unusual network activity and using the DataDog’s monitoring functionality to alert when an anomaly is detected and potentially trigger scripts to take action (like block an IP address causing the anomaly.






