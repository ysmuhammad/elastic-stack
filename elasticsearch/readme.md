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

#detail the private IPs of all elasticsearch master nodes:
discovery.seed_hosts: ["<IP_ADDRESS>"]

#Uncomment and set 2 if you have 3 masters
discovery.zen.minimum_master_nodes: 2

#Add the hostname of each node if you have more than 1 nodes
cluster.initial_master_nodes: ["master1","master2","master3"]

" >> /data/elasticsearch/instelk/etc/config/elasticsearch.yml
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
