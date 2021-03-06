# NoSQL

* [Cassandra](#cassandra)
  * [Configuration](#cassandra---configuration)
    * [Optimization]
    * [Clustering]
* MongoDB
* Apache Phoneix


# Cassandra


## Cassandra - Configuration

RHEL Install [1][2]:
```
# yum install java-1.8.0-openjdk
# vim /etc/yum/repos.d/datastax.repo
[datastax]
name = DataStax Repo for Apache Cassandra
baseurl = http://rpm.datastax.com/community
enabled = 1
gpgcheck = 0
# yum install cassandra30
# systemctl daemon-reload
```

Configuration options:
* cluster_name = The unique name for a cluster.
  * (Default: Test Cluster)
* listen_address = The IP address to listen on for clustering.
  * (Default: localhost)
* listen_interface = The network interface to listen on for clustering.
  * (Default: eth0)
* rpc_address = The IP address to listen on for client requests.
  * (Default: localhost)
* rpc_interface = The network interface to listen on for client requests.
  * (Default: eth1)
* start_rpc = Start the client service to allow incoming connections.
  * (Default: false)
* disk_optimization_strategy = Specify the type of disk to optimize reads/writes for.
  * ssd = SSDs.
  * spinning = Spinning disk hard drives.
  * (Default: ssd)
* disk_failure_policy = The action to take when a disk is missing or in a failed state.
  * die = Kill off all processes.
  * stop = Gracefully stop the service.
  * best_effort = Do not use the disk but attempt to respond to requests with any data available.
  * ignore = Ignore any major I/O errors and provide failure responses to any requests.
  * (Default: stop)
* endpoint_snitch = Select a snitch interface for clustering.
  * SimpleSnitch =  Cluster based on proximity, but datacenter and rack location does not matter. Recommended for clusters in one region.
  * GossipingPropertyFileSnitch = Cluster based on the datacenter and rack location. Recommended for a multidatacenter cluster.
  * Ec2Snitch = Cluster based on Amazon EC2 regions and compute availability zones.
  * Ec2MultiRegionSnitch = Allows multiple Amazon EC2 regions to be used via public floating IPs.
  * RackInferringSnitch = Similar to GossipingPropertyFileSnitch except that the datacenter is automatically determined by the 2nd octet of the IP and the rack is determined by the 3rd.
  * GoogleCloudSnitch = Cluster based on the Google Cloud Platform's regions and compute availability zones.
  * CloudstackSnitch = Integrate with the Apache Cloudstack.
* seed_provider = The IP addresses of Cassandra servers in other datacenters to replicate to. At least one node should be a seed provider in every datacenter. Not all nodes should be seed providers due to that leading to performance issues.
  * \- class_name: org.apache.cassandra.locator.SimpleSeedProvider
    * parameters:
      * \- seeds: "`<IP_ADDRESS_1>`", "`<IP_ADDRESS_2>`"
* concurrent_reads
  * (Default: 32)
  * (Recommended: 16 * `<COUNT_OF_DISKS>`)
* concurrent_writes
  * (Default: 32)
  * (Recommended: 16 * `<COUNT_OF_CPUS>`)
* concurrent_counter_writes
  * (Default: 32)
  * (Recommended: 16 * `<COUNT_OF_DISKS>`)
* concurrent_batchlog_writes
  * (Default: 32)
  * (Recommended: 16 * `<COUNT_OF_CPUS>`)
* concurrent_materialized_view_writes
  * (Default: 32)
  * (Recommended: Less than the concurrent reads/writes)
* incremental_backups = Choose whether or not to use incremental backups. When taking snapshots, hardlinks will be used to refer back to old data for efficient backups.
  * (Default: false)
* snapshot_before_compact = Choose whether or not to automatically take backups before running a compaction.
  * (Default: false)

[3]

1. "How To Install Cassandra on CentOS 7" liquidweb Knowledgebase. Accessed October 16, 2016. https://www.liquidweb.com/kb/how-to-install-cassandra-on-centos-7/
2. "Installing the DataStax Distribution of Apache Cassandra 3.x on RHEL-based systems." DataStax Distribution of Apache Cassandra 3 Documentation. October 14, 2016. Accessed October 16, 2016. http://docs.datastax.com/en/cassandra/3.x/cassandra/install/installRHEL.html

3. http://docs.datastax.com/en/cassandra/3.0/cassandra/configuration/configCassandra_yaml.html


### Cassandra - Configuration - Optimization

http://www.slideshare.net/planetcassandra/datastax-extreme-cassandra-optimization-the-sequel


### Cassandra - Configuration - Clustering

https://www.digitalocean.com/community/tutorials/how-to-run-a-multi-node-cluster-database-with-cassandra-on-ubuntu-14-04