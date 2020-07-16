## Elasticsearch v7.4.2 Installation Manual

##### Component

- 3 Nodes using RHEL 7.8

- elasticsearch-7.4.2-linux-x86_64.tar.gz



### 1.) Prepare User
- Run as root.
- Execute command on all nodes.
##### 1.1) Create User for Elasticsearch
Exec command:
```shell
groupadd dbgrp -g 3000

useradd instelk -u 3112 -s /bin/bash -m -d /home/instelk -g dbgrp
echo -e "instelk\ninstelk" | passwd instelk
chage -I -1 -m 0 -M 99999 -E -1 instelk
```

### 2.) Install Elasticsearch from Source
- Run as root
- Execute command on all nodes.
##### 2.1) Create Source Folder
Exec command:
```shell
mkdir /Source
chmod 777 /Source
```
##### 2.2) Download Elasticsearch Source
Exec command:
```shell
cd /Source
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.2-linux-x86_64.tar.gz

tar -zxvf elasticsearch-7.4.2-linux-x86_64.tar.gz 

chown -R instelk:dbgrp /Source/elasticsearch-7.4.2/
```
##### 2.3) Create Service Directory
Exec command:
```shell
mkdir /data
chmod 755 /data

mkdir -p /data/elasticsearch/instelk/db /data/elasticsearch/instelk/etc /data/elasticsearch/instelk/logs

chown instelk:dbgrp /data/ -R
```

##### 2.4) Move Binary & Config Files to The Service Directory
Exec command:
```shell
su - instelk
cp -r /Source/elasticsearch-7.4.2/* /data/elasticsearch/instelk/etc/
```



### 3.) Elasticsearch Parameter Tuning
- Run as instelk user
##### 3.1) Modify elasticsearch.yml File
Execute this command on Node 01 :
```shell
echo "#action.destructive_requires_name: true
cluster.name: lisa-uis

#give your nodes a name (change node number from node to node).
node.name: "master1"

#define node 1 as master-eligible:
node.master: true

#define nodes 2 and 3 as data nodes:
node.data: true

path.data: /data/elasticsearch/instelk/db
path.logs: /data/elasticsearch/instelk/logs

#enter the private IP and port of your node:
network.host: <IP_ADDRESS>
http.port: 9200

#detail the private IPs of your nodes:
discovery.seed_hosts: ["<IP_ADDRESS>"]

#Uncomment and set 2 if you have 3 masters
discovery.zen.minimum_master_nodes: 2

#Add the hostname of each node if you have more than 1 nodes
cluster.initial_master_nodes: ["master1","master2","master3"]

" >> /data/elasticsearch/instelk/etc/config/elasticsearch.yml
```



Execute this command on Node 02 :
```shell
echo "#action.destructive_requires_name: true
cluster.name: lisa-uis

#give your nodes a name (change node number from node to node).
node.name: "master2"

#define node 1 as master-eligible:
node.master: true

#define nodes 2 and 3 as data nodes:
node.data: true

path.data: /data/elasticsearch/instelk/db
path.logs: /data/elasticsearch/instelk/logs

#enter the private IP and port of your node:
network.host: <IP_ADDRESS>
http.port: 9200

#detail the private IPs of your nodes:
discovery.seed_hosts: ["<IP_ADDRESS>"]

#Uncomment and set 2 if you have 3 masters
discovery.zen.minimum_master_nodes: 2

#Add the hostname of each node if you have more than 1 nodes
cluster.initial_master_nodes: ["master1","master2","master3"]

" >> /data/elasticsearch/instelk/etc/config/elasticsearch.yml
```


Execute this command on Node 03 :
```shell
echo "#action.destructive_requires_name: true
cluster.name: lisa-uis

#give your nodes a name (change node number from node to node).
node.name: "master3"

#define node 1 as master-eligible:
node.master: true

#define nodes 2 and 3 as data nodes:
node.data: true

path.data: /data/elasticsearch/instelk/db
path.logs: /data/elasticsearch/instelk/logs

#enter the private IP and port of your node:
network.host: <IP_ADDRESS>
http.port: 9200

#detail the private IPs of your nodes:
discovery.seed_hosts: ["<IP_ADDRESS>"]

#Uncomment and set 2 if you have 3 masters
discovery.zen.minimum_master_nodes: 2

#Add the hostname of each node if you have more than 1 nodes
cluster.initial_master_nodes: ["master1","master2","master3"]

" >> /data/elasticsearch/instelk/etc/config/elasticsearch.yml
```




##### 3.2) Set Elasticsearch JVM

Exec command on all nodes:
```shell
sed -i "s/-Xms1g/-Xms8g/g" /data/elasticsearch/instelk/etc/config/jvm.options
sed -i "s/-Xmx1g/-Xmx8g/g" /data/elasticsearch/instelk/etc/config/jvm.options
exit
```

### 4.) Create SystemD
- Run as root
- Execute on all nodes
##### 4.1) Create SystemD Script
Exec command:
```shell
echo "[Unit]
Description=Elasticsearch
Documentation=http://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
RuntimeDirectory=elasticsearch
PrivateTmp=true
Environment=ES_HOME=/data/elasticsearch/instelk/etc
Environment=ES_PATH_CONF=/data/elasticsearch/instelk/etc/config
Environment=PID_DIR=/data/elasticsearch/instelk/etc

WorkingDirectory=/data/elasticsearch/instelk/etc

User=instelk
Group=dbgrp

ExecStart=/data/elasticsearch/instelk/etc/bin/elasticsearch -p /data/elasticsearch/instelk/etc/elasticsearch.pid --quiet

# StandardOutput is configured to redirect to journalctl since
# some error messages may be logged in standard output before
# elasticsearch logging system is initialized. Elasticsearch
# stores its logs in /var/log/elasticsearch and does not use
# journalctl by default. If you also want to enable journalctl
# logging, you can simply remove the "quiet" option from ExecStart.
StandardOutput=journal
StandardError=inherit

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65535

# Specifies the maximum number of processes
LimitNPROC=65535

# Specifies the maximum size of virtual memory
LimitAS=infinity

# Specifies the maximum file size
LimitFSIZE=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=0

# SIGTERM signal is used to stop the Java process
KillSignal=SIGTERM

# Send the signal only to the JVM rather than its control group
KillMode=process

# Java process is never killed
SendSIGKILL=no

# When a JVM receives a SIGTERM signal it exits with code 143
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target

# Built for packages-7.4.0 (packages)
" >> /etc/systemd/system/elasticsearch.service
```






##### 4.2) Adjust Kernel Virtual Memory Size

Exec command:

```shell
sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
```

##### 4.3) Adjust Kernel Max Opened Files & Max Number of Processes
Exec command:
```shell
echo "
*       hard    nofile  65535
*       soft    nofile  65535" >> /etc/security/limits.conf

echo "
*       hard    nproc   65535
*       soft    nproc   65535" >> /etc/security/limits.conf
```





##### 4.4) Start Elasticsearch

```shell
vi /data/elasticsearch/instelk/etc/config/jvm.options
```



Exec command:

```shell
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
```

### 5.) Check Elasticsearch
- Run as any user.
##### 5.1) Check Elasticsearch Cluster Status
Exec command:
```shell
curl http://<IP_ADDRESS>:9200/_cluster/health
```
