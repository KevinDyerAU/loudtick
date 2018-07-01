# loudtick

#####################################################################################################################################################################################################################################################################
INSTALL AND SETUP DOCKER to WINDOWS HOST
#####################################################################################################################################################################################################################################################################

################################################################################################################################
## Portainer
docker volume create portainer_data
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

user: admin
pass: password

################################################################################################################################
##LOUDML Community
docker pull loudml/community
docker run -ti -p 8077:8077 -v c:/java_rt/loudml:/etc/loudml  -v c:/java_rt/loudml/lib:/var/lib/loudml:rw loudml/community


apt-get update
apt-get install iputils-ping ## install ping
apt-get install net-tools ## install ifconfig
apt-get install curl


##Find Docker Host
netstat -nr | grep '^0\.0\.0\.0' | awk '{print $2}'  =>> Actually should be the DNS host.docker.internal

## List host dir after mounting
docker run --rm -v c:/Users:/data alpine ls /data


https://medium.com/swlh/easy-machine-learning-tricks-for-linux-developers-in-2018-f697d153d904
Loud ML documentation is available on Github (https://github.com/regel/loudml) and online at www.loudml.io/guide
Also, you can watch the Apr 17 video recording for an interactive demo: https://www.influxdata.com/resources/build-your-a-predictive-model-with-influxdb-and-loud-ml/

## EL7
$ yum install loudml loudml-influx

##Docker
https://github.com/lmangani/loudml-docker


After installation, find the configuration file located in /etc/loudml/config.yml

You must declare the data sources i.e. where to read the data and how to connect to NoSQL databases.
The Loud ML 1.1 beta release already supports popular databases InfluxDb and Elasticsearch.
To configure InfluxDb, define the name and address to connect to the database:
---
storage:
  path: /var/lib/loudml
server:
  listen: localhost:8077
datasources: 
 - name: influx
   type: influxdb
   addr: host.docker.internal:8086
   database: telegraf
	
#####################################################################################################################################################################################################################################################################
POST INSTALL
#####################################################################################################################################################################################################################################################################
###Your first predictive model
Say your InfluxDb database (or your Elasticsearch indexes) contains CPU measurements for the server hosting your web application, 
let’s name them cpu_load, and you have 30 days of history with 1 minute resolution.

Your first model will predict a single feature avg_cpu_load, and will:

average data over five-minute intervals (the bucket_interval); and
assume the last three bucket intervals (span=3, so, in total 15 minutes) will be used to guess the next cpu_load value.


Let’s create this model using the CLI.

1) You must write the file that describe your model. This file can be either JSON or YAML.
	We will define in model.yml file a single ‘feature’ to learn the shape of the average cpu load using 5 minutes bucket intervals:
	---
		name: my-timeseries-model
		type: timeseries
		default_datasource: my-influx-datasource
		# Size of buckets for data aggregation and prediction 
		bucket_interval: 5m
		# Number of preceding buckets required to predict the next bucket
		span: 3
		interval: 60s
		offset: 30
		max_evals: 5
		features:
		- name: avg_cpu_load
			measurement: system
			metric: avg
			field: cpu_load
			default: 0

2) let’s create this first model:
		$ loudml create-model model.yml			
			
			
	KD NOTE: Use Portainer to ssh to docker instance


#####################################################################################################################
###The Kung fu lesson is about to begin. 

By training this model, it will learn how the data evolves over time. You can think of deep learning as a way to approximate *any* function.
3) The model training is launched with the following command:
		$ loudml train <model_name> --from <from_date> --to <to_date>	
			
	Accepted date formats are:
	UNIX timestamp in seconds
	ISO 8601 format, example: 2018–01–26T16:47:25Z
	Relative date, example: now-20s, now-45m, now-3h, now-1d, now-3w…
	We will train the above model using a 30 days history:				
		$ loudml train my-timeseries-model --from now-30d --to now
		$ loudml train avg_price-model --from now-180d --to now
		
	When it’s done, the command will report the actual error loss: the lower the better!
	To show additional model information, you can run:
		$ loudml show-model <model_name>	
			
			
			
#####################################################################################################################
###It is show time! Predictive capabilities

You know Kung fu, so let’s practice. You’ve trained a model. 
4) You can make this model output predictions for avg_cpu_load on a regular interval. 
	This output may be written to another data source, your application, or merely stdout, and compared to the actual values for anomaly detection.

	You can enable the loudmld service to execute in the background; if you are running EL7 the system command will be:
	$ systemctl enable loudmld.service
	$ systemctl start loudmld.service			
				
			
The loudmld process exposes a HTTP API that you can control using curl. To start predicting future avg_cpu_load values, you can issue this curl command:

		curl http://localhost:8077/models/my-timeseries-model/_start			
			
	KD NOTE: Pick host from Portainer
			
It will tell the loudmld process to wake up at periodic intervals (interval in config.yml - don’t confuse with bucket_interval), 
pull the data from your data source, and finally output the predicted values to the time series database.			
			
			
			
If you’re using InfluxDb, you can visualize the result using Chronograf. 
The prediction is stored in the same database under the measurement called prediction_<model_name> and must be displayed using a GROUB BY time(bucket_interval) query.

Hopefully, this first Kung fu lesson has been a success!

Loud ML documentation is available on Github (https://github.com/regel/loudml) and online at www.loudml.io/guide

Also, you can watch the Apr 17 video recording for an interactive demo: https://www.influxdata.com/resources/build-your-a-predictive-model-with-influxdb-and-loud-ml/			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			
			