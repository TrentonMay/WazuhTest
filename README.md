# Documentation steps to install the necessary components for the Bounty Hunt!

## Prerequisits:
To start this hunt you're going to need too build a hub, a entry, an exit, and two blank servers. 

## Wazuh Server
This service will be built on your first blank server **NOT THE SECOND**.

###### You can follow along with the documentation steps here
(https://documentation.wazuh.com/current/installation-guide/index.html)

###### SSH into your blank server and begin running the following commands

###### Be sure that everything is updated properly and ensure required packages are installed
```
apt-get update
apt-get install curl apt-transport-https lsb-release
```

###### Let's check to see if our python symlink is correct
```
if [ ! -f /usr/bin/python ]; then ln -s /usr/bin/python3 /usr/bin/python; fi
```

###### Install the GPG key
```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
```

###### Add the required repository
```
echo "deb https://packages.wazuh.com/3.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```

###### Update
```
apt-get update
```

###### Wazuh manager Installation
```
apt-get install wazuh-manager
```

###### Let's see if the installation went well and the service is running
```
systemctl status wazuh-manager
```

###### Installing NodeJs
```
curl -sL https://deb.nodesource.com/setup_8.x | bash -
apt-get install nodejs
```

###### After we install NodeJs lets make sure everything went well and run this command
###### This will let us know if the Node Package Manager installed correctly 
```
npm
```

###### Install the Wazuh API
```
apt-get install wazuh-api
```

###### Check the status of the API 
```
systemctl status wazuh-api
```

## Installing the Elastic Stack
```
add-apt-repository ppa:webupd8team/java
apt-get update
apt-get install oracle-java8-installer
```

###### Install the Repo and the GPG key
```
apt-get install curl apt-transport-http
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-6.x.list
apt-get update
```

###### Install the ElasticSearch package
```
apt-get install elasticsearch=6.4.1
```

###### Enable the service
```
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```

###### To see if the ElasticSearch has started run this command
```
curl "localhost:9200/?pretty"
```

###### You should see this pop up
```
{
  "name" : "Zr2Shu_",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "M-W_RznZRA-CXykh_oJsCQ",
  "version" : {
    "number" : "6.4.1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "053779d",
    "build_date" : "2018-07-20T05:20:23.451332Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

###### Load the Wazuh template for ElasticSearch
```
curl https://raw.githubusercontent.com/wazuh/wazuh/3.6/extensions/elasticsearch/wazuh-elastic6-template-alerts.json | curl -XPUT 'http://localhost:9200/_template/wazuh' -H 'Content-Type: application/json' -d @-
```

## Installing Logstash

###### Install the Package
```
apt-get install logstash=1:6.4.1-1
```

###### Install the Wazuh configuration for Logstash
```
curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/3.6/extensions/logstash/01-wazuh-local.conf
```

###### Allow Logstash to read the alerts.json file
```
usermod -a -G ossec logstash
```

###### Enable the Logstash service
```
systemctl daemon-reload
systemctl enable logstash.service
systemctl start logstash.service
```

## Install Kibana

###### Install the Kibana Package
```
apt-get install kibana=6.4.1
```

###### Install the Wazuh plugin for Kibana
```
export NODE_OPTIONS="--max-old-space-size=3072"
```

###### Install the Wazuh app
```
sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp-3.6.1_6.4.1.zip
```

###### Next we need to edit the Kibana configuration file to allow it to accept connections
```
vi /etc/kibana/kibana.yml
```

###### Uncomment the server.host setting and make sure it says
`"0.0.0.0"`

###### Start the Kibana service
```
systemctl daemon-reload
systemctl enable kibana.service
systemctl start kibana.service
```

###### Disable the ElasticStack repo and update
```
sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/elastic-6.x.list
apt-get update
```

## Install the Wazuh Agent server

###### Spin up another blank server. When it is finished building ssh into the server. 

###### Install the Wazuh GPG key
```
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
```

###### Add the Repository. Then update
```
echo "deb https://packages.wazuh.com/3.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
apt-get update
```

###### Install Wazuh Agent
```
apt-get install wazuh-agent
```

