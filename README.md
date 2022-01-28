# Running Elasticsearch on The NGD Systems Computational Storage Drives  “CSDs”

# What is Elasticsearch? 
Elasticsearch is a distributed, open source search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured. Known for its simple REST APIs, distributed nature, speed, and scalability, Elasticsearch is the central component of the Elastic Stack, a set of open source tools for data ingestion, enrichment, storage, analysis, and visualization.

# Elasticsearch & NGD Computational Storage Drives
Here our goal is to highlight how taking a distributed Elasticsearch toolset and distributing across NGD Computational Storage Drives can provide the necessary support to increase performance and reduce system complexity to accomplish the goal of proper data management. Using Computational Storage Drives to offload the initial work required, you can substantially reduce the amount of round robin data movement and time associated with it to get the desired results. If we take the opportunity to minimize the data transferred from the disk to the DRAM and CPU the efficiency of the system increases while the complexity of the platform is reduced. Since there are no ‘new’ components required, like GPU or FPGA accelerators, and storage is needed to hold the data being gathered and searched locally already, the use of CSDs from NGD Systems does not increase system design.

To understand how we can accomplish this task, we need to look at the configuration of the Elasticsearch solution on a given server platform. In this instance, we showcase how many servers may be required to accomplish a deployment of the search tools. By choosing to offload portions of the platform to the devices we can provide more value to the overall system Total Cost of Ownership (TCO). Focusing on the best use of the system resources for search versus other needed activities of the system. 

![image](https://user-images.githubusercontent.com/31414094/139505253-836667e0-bed5-4e96-bb52-e808b1a6e7c7.png)

What does this new architecture look like, Well in our example we reconfigure the platform into only the needed number of servers for resiliency and provide CPU resources on the NGD Systems Computational Storage Drives (CSDs) by overlaying the Data and Master nodes and releasing the added CPUs and servers in the system to be used for other tasks.


# Because Elasticsearch uses Java, we need to ensure the Java Development Kit (JDK) is installed. 
As NGD CSD operating system is a Linux aarch64 (64-bit ARM) systems:
```
ngd@node1:~$ sudo bash
root@node1:~$ cd /usr/lib/jvm
root@node1:/usr/lib/jvm$ wget https://download.oracle.com/java/17/latest/jdk-17_linux-aarch64_bin.tar.gz
root@node1:/usr/lib/jvm$ tar zxvf jdk-17_linux-aarch64_bin.tar.gz
root@node1:/usr/lib/jvm$ update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk-17.0.1/bin/java 17
root@node1:/usr/lib/jvm$ exit
ngd@node1:~$ sudo update-alternatives --config java
```
In the above menu, select the biggest selection number, which refers to JDK-18.

# Installing Elasticsearch on NGD CSD

```
ngd@node1:~$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.1-arm64.deb
ngd@node1:~$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.1-arm64.deb.sha512
ngd@node1:~$ shasum -a 512 -c elasticsearch-7.15.1-arm64.deb.sha512
ngd@node1:~$ sudo dpkg -i elasticsearch-7.15.1-arm64.deb
```

# Running Elasticsearch with "systemd"
To configure Elasticsearch to start automatically when the system boots up, run the following commands:

1-   Change "elasticsearch.service" config file as follows:
```
sudo vim /usr/lib/systemd/system/elasticsearch.service
```

```
[Unit]
Description=Elasticsearch
Documentation=https://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
Type=notify
RuntimeDirectory=elasticsearch
PrivateTmp=true
Environment=ES_HOME=/usr/share/elasticsearch
Environment=ES_PATH_CONF=/etc/elasticsearch
Environment=PID_DIR=/var/run/elasticsearch
Environment=ES_SD_NOTIFY=true
EnvironmentFile=-/etc/default/elasticsearch

WorkingDirectory=/usr/share/elasticsearch

User=elasticsearch
Group=elasticsearch

ExecStart=/usr/share/elasticsearch/bin/systemd-entrypoint -p ${PID_DIR}/elasticsearch.pid --quiet
#ExecStart=/usr/share/elasticsearch/bin/systemd-entrypoint
#ExecStart=/usr/share/elasticsearch/bin/elasticsearch

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
LimitNPROC=4096

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

# Allow a slow startup before the systemd notifier module kicks in to extend the timeout
TimeoutStartSec=180

[Install]
WantedBy=multi-user.target

# Built for packages-7.15.1 (packages)

```


```

We only changed the value of TimeoutStartSec to 180. 

ngd@node1:~$ sudo /bin/systemctl daemon-reload
ngd@node1:~$ sudo /bin/systemctl enable elasticsearch.service
```

2- Disable xpack security by add “xpack.security.enabled: false” to /etc/elasticsearch/elasticsearch.yml 
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

Elasticsearch can be started as follows:

```
ngd@node1:~$ sudo systemctl start elasticsearch.service
```

#Checking that Elasticsearch is runningedit

Now you can test that your Elasticsearch node is running by sending an HTTP request to port 9200 on localhost:
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

Elasticsearch can be stopped as follows:

```
ngd@node1:~$ sudo systemctl stop elasticsearch.service
```








