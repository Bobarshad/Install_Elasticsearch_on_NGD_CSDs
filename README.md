# Running Elasticsearch on The NGD Systems Computational Storage Drives  “CSDs”

# What is Elasticsearch? 
Elasticsearch is a distributed, open source search and analytics engine for all types of data, including textual, numerical, geospatial, structured, and unstructured. Known for its simple REST APIs, distributed nature, speed, and scalability, Elasticsearch is the central component of the Elastic Stack, a set of open source tools for data ingestion, enrichment, storage, analysis, and visualization.

# Elasticsearch & NGD Computational Storage Drives
Here our goal is to highlight how taking a distributed Elasticsearch toolset and distributing across NGD Computational Storage Drives can provide the necessary support to increase performance and reduce system complexity to accomplish the goal of proper data management. Using Computational Storage Drives to offload the initial work required, you can substantially reduce the amount of round robin data movement and time associated with it to get the desired results. If we take the opportunity to minimize the data transferred from the disk to the DRAM and CPU the efficiency of the system increases while the complexity of the platform is reduced. Since there are no ‘new’ components required, like GPU or FPGA accelerators, and storage is needed to hold the data being gathered and searched locally already, the use of CSDs from NGD Systems does not increase system design.

To understand how we can accomplish this task, we need to look at the configuration of the Elasticsearch solution on a given server platform. In this instance, we showcase how many servers may be required to accomplish a deployment of the search tools. By choosing to offload portions of the platform to the devices we can provide more value to the overall system Total Cost of Ownership (TCO). Focusing on the best use of the system resources for search versus other needed activities of the system. 

![image](https://user-images.githubusercontent.com/31414094/139505253-836667e0-bed5-4e96-bb52-e808b1a6e7c7.png)
