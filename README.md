# elastic-stack
this is configuration of logstash and filebeat file

## installation and configuration of elasticsearch
install elastic stack on ubuntu :

step 1 : you must add elastic repository with this command
```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch |sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
```
step 2:
```bash
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```
step 3:
```bash
sudo apt update
```

step 4:
```bash
sudo apt install elasticsearch
```
by default the elasticsearch.yaml file is in the current location and configured on localhost:9200. if you want to change it you must go to this link and change the port anf ip.
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

## Note:
Elasticsearch’s configuration file is in YAML format, which means that we need to maintain the indentation format. Be sure that you do not add any extra spaces as you edit this file.

step 5: to install elastic search run this command
```bash
sudo systemctl start elasticsearch
```
step 5: 
run the following command to enable Elasticsearch to start up every time your server boots:
```bash
sudo systemctl enable elasticsearch
```

step 6: 
You can test whether your Elasticsearch service is running by sending an HTTP request:
```bash
curl -X GET "localhost:9200"
```
the response is something like this:
```bash
Output
{
  "name" : "Elasticsearch",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "n8Qu5CjWSmyIXBzRXK-j4A",
  "version" : {
    "number" : "7.17.2",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "de7261de50d90919ae53b0eff9413fd7e5307301",
    "build_date" : "2022-03-28T15:12:21.446567561Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## Installing and Configuring the Kibana Dashboard
step 1: 
```bash
sudo apt install kibana
```
step 2: Then enable and start the Kibana service
```bash
sudo systemctl enable kibana
```
```bash
sudo systemctl start kibana
```

## Installing and Configuring Logstash (Important)

step 1:Install Logstash with this command
```bash
sudo apt install logstash
```
## Note : the place that you must put your logstash configuration files is here: 
```bash
/etc/logstash/conf.d
```
step 2: make a file in this directory to put your config in it 
```bash
sudo nano /etc/logstash/conf.d/logstash.conf
```
step 3: put this configs in this file and save it by ctrl + x and then y
```bash
input {
   beats {
     port => 5044
   }
}
output {
   if[fields][service_name] == "multipay"{

   elasticsearch {
     hosts => ["localhost:9200"]
     index => "multipay-%{+yyyy.MM.dd}"
   }
   stdout{
     codec => rubydebug
   }

 }else if[fields][service_name] == "inquiry"{

   elasticsearch {
     hosts => ["localhost:9200"]
     index => "inquiry-%{+yyyy.MM.dd}"

   }
   stdout{
     codec => rubydebug
   }

}

}
```

step 4: to ensure that your config file is correct run this command
```bash
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
```
## Note: 
if the config file be Ok you will get the OK response

step 5: start logstash
```bash
sudo systemctl start logstash
```
step 6: 
```bash
sudo systemctl enable logstash
```
## Installing and Configuring Filebeat

filebeat read the log files from any address that you want and send it to port 5044 that logstash is up on this port

step 1:
```bash
sudo apt install filebeat
```
## Note: the configuration of the filebeat is in this address : /etc/filebeat/filebeat.yml
step2: edit configuration file (filebeat.yml)
```bash
sudo nano /etc/filebeat/filebeat.yml
```
step 3: put following configs in this file
```bash
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - '/home/bizhan/logs/multipay.log'
  fields_under_root: false
  fields:
    service_name: "multipay"
- type: log
  enabled: true
  paths:
    - '/home/bizhan/logs/inquiry.log'
  fields_under_root: false
  fields:
    service_name: "inquiry"

output.logstash:
  hosts: ["localhost:5044"]
```
step 4: to load the template run this command
```bash
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["localhost:9200"]'
```
step 5: to make connection with kibana use this command
```bash
sudo filebeat setup -E output.logstash.enabled=false -E output.elasticsearch.hosts=['localhost:9200'] -E setup.kibana.host=localhost:5601
```
step 6: to run the filebeat run this command
```bash
sudo systemctl start filebeat
```
```bash
sudo systemctl enable filebeat
```
## Conclusion
```python
a brief and precision explanation about every parts of Elastic stack:

1.Elasticsearch: a noSql database that stores the logs. its on port 9200
2.kibana: gives a GUI and an interface and dashboard that you can see the datas in it. its on port 5601
3.logstash: logstash is a service that gives the log datas from filebeats and filter and send it to the elasticsearch for storing. it is on port 5044
4.filebeat: filebeat is a service that read the log files from any address and send their datas to the logstash.
```
