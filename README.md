# Running Elasticsearch on The NGD Systems Computational Storage Drives  “CSDs”

# What is Elasticsearch? 
Elasticsearch is a distributed, open source search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured. Known for its simple REST APIs, distributed nature, speed, and scalability, Elasticsearch is the central component of the Elastic Stack, a set of open source tools for data ingestion, enrichment, storage, analysis, and visualization.

# Elasticsearch & NGD Computational Storage Drives
Here our goal is to highlight how taking a distributed Elasticsearch toolset and distributing across NGD Computational Storage Drives can provide the necessary support to increase performance and reduce system complexity to accomplish the goal of proper data management. Using Computational Storage Drives to offload the initial work required, you can substantially reduce the amount of round robin data movement and time associated with it to get the desired results. If we take the opportunity to minimize the data transferred from the disk to the DRAM and CPU the efficiency of the system increases while the complexity of the platform is reduced. Since there are no ‘new’ components required, like GPU or FPGA accelerators, and storage is needed to hold the data being gathered and searched locally already, the use of CSDs from NGD Systems does not increase system design.

To understand how we can accomplish this task, we need to look at the configuration of the Elasticsearch solution on a given server platform. In this instance, we showcase how many servers may be required to accomplish a deployment of the search tools. By choosing to offload portions of the platform to the devices we can provide more value to the overall system Total Cost of Ownership (TCO). Focusing on the best use of the system resources for search versus other needed activities of the system. 

![image](https://user-images.githubusercontent.com/31414094/139505253-836667e0-bed5-4e96-bb52-e808b1a6e7c7.png)

What does this new architecture look like, Well in our example we reconfigure the platform into only the needed number of servers for resiliency and provide CPU resources on the NGD Systems Computational Storage Drives (CSDs) by overlaying the Data and Master nodes and releasing the added CPUs and servers in the system to be used for other tasks.

# Installing Elasticsearch on NGD CSD

```
ngd@node1:~$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.1-arm64.deb
ngd@node1:~$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.1-arm64.deb.sha512
ngd@node1:~$ shasum -a 512 -c elasticsearch-7.15.1-arm64.deb.sha512
ngd@node1:~$ sudo dpkg -i elasticsearch-7.15.1-arm64.deb
```

# Because Elasticsearch uses Java, we need to ensure the Java Development Kit (JDK) is installed. 
As NGD CSD operating system is a Linux aarch64 (64-bit ARM) systems:
```
ngd@node1:~$ sudo bash
root@node1:~$ cd /usr/lib/jvm
root@node1:/usr/lib/jvm$ wget https://download.java.net/java/early_access/jdk18/21/GPL/openjdk-18-ea+21_linux-aarch64_bin.tar.gz
root@node1:/usr/lib/jvm$ tar zxvf openjdk-18-ea+21_linux-aarch64_bin.tar.gz
root@node1:/usr/lib/jvm$ ln -s /etc/alternatives/java /usr/bin/java
root@node1:/usr/lib/jvm$ sudo update-alternatives --install /usr/bin/jave java /usr/lib/jvm/jdk-18/bin/java 18
root@node1:/usr/lib/jvm$ exit
ngd@node1:~$ sudo update-alternatives --config java
```
In the above menu, select the biggest selection number, which refers to JDK-18.

# Running Elasticsearch with "systemd"
To configure Elasticsearch to start automatically when the system boots up, run the following commands:

```
ngd@node1:~$ sudo /bin/systemctl daemon-reload
ngd@node1:~$ sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch can be started and stopped as follows:

```
ngd@node1:~$ sudo systemctl start elasticsearch.service
ngd@node1:~$ sudo systemctl stop elasticsearch.service
```

#Checking that Elasticsearch is runningedit
1- Disable xpack security by add “xpack.security.enabled: false” to /etc/elasticsearch/elasticsearch.yml 
```
sudo vim /etc/elasticsearch/elasticsearch.yml
```

```
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#
#cluster.name: my-application
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
#node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/lib/elasticsearch
#
# Path to log files:
#
path.logs: /var/log/elasticsearch
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
#
#network.host: 192.168.0.1
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
xpack.security.enabled: false
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Allow wildcard deletion of indices:
#
#action.destructive_requires_name: false

```

2- Now you can test that your Elasticsearch node is running by sending an HTTP request to port 9200 on localhost:
```
ngd@node1:~$ curl http://localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "elasticsearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

```








