# 4.0 Designing, Troubleshooting, and Integrating Systems – 40% Weight

* [Brokers and Zookeeper](#brokers-and-zookeeper)
    * [CPU, RAM, Network, Storage Considerations](#cpu-ram-network-storage-considerations)
* [Number of Nodes](#number-of-nodes)
* [Rack Awareness](#rack-awareness)
* [Kafka Connect](#kafka-connect)
    * Source and Sink Connectors
* [Scalability and High Availability](#scalability-and-high-availability)
* [Business Continuity / DR](#business-continuity--dr)
* [Data Retention](#data-retention)

## Brokers and Zookeeper

### Zookeeper

* [📺 Zookeeper Configuration](https://www.udemy.com/course/kafka-cluster-setup/)
* [Zookeeper Configuration File Example](./../Apache_Kafka_Series_Kafka_Cluster_Setup_Administration/course_resources/zookeeper/zookeeper.properties)
* [Zookeeper with Kafka](https://learn.conduktor.io/kafka/zookeeper-with-kafka/)
* [Zookeeper Troubleshooting](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_troubleshooting)
* [Zookeeper Design Deployment](https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_designing)
- Kafka utilizes Zookeeper for storing metadata information about the brokers, topics,
and partitions.
- It's not a good practise to share the Zookeeper ensemble with other applications.
- Kafka is sensitive
  to Zookeeper latency and timeouts, and an interruption in communications with the
  ensemble will cause the brokers to behave unpredictably.
- This can easily cause multi‐
  ple brokers to go offline at the same time, should they lose Zookeeper connections,
  which will result in offline partitions

### Broker

#### Adding a new Broker. What will happen?

- We can add Brokers to scale
- New added Brokers don't have any topic assign.
- Topics Partition Rebalance operation is required to spread the load equally in the cluster. 

#### Replace a existing Broker keeping the existing data
- We terminate the Broker instance
- Create a new Broker instance
- Attach the available EBS to the new instance
- Start Kafka service on the new Broker instance
- That new instance, ONLY will replicate the data that doesn't have yet.

#### Terminate a Broker definitively
- We need to first move all the current partitions from the terminated Broker to other existing Brokers.
- Onece done, we can terminate the instance.

#### Configuration Management
- `read-only` configuration requires broker restart
    - Change the configuration on `server.properties`
        - Kafka uses key-value pairs in the property file format for configuration. These values can be supplied either from a file or programmatically.
    - Rolling restart of the brokers
- https://kafka.apache.org/documentation/#dynamicbrokerconfigs
- [Broker Configs](https://kafka.apache.org/documentation/#brokerconfigs)
    - Important parameters
        - [auto.create.topics.enable](https://kafka.apache.org/documentation/#brokerconfigs_auto.create.topics.enable)
        - [background.threads](https://kafka.apache.org/documentation/#brokerconfigs_background.threads)
        - [delete.topic.enable](https://kafka.apache.org/documentation/#brokerconfigs_delete.topic.enable)
        - [log.flush.interval.messages](https://kafka.apache.org/documentation/#brokerconfigs_log.flush.interval.messages)
        - [log.retention.hours](https://kafka.apache.org/documentation/#brokerconfigs_log.retention.hours)
        - [message.max.bytes](https://kafka.apache.org/documentation/#brokerconfigs_message.max.bytes)
        - [min.insync.replicas](https://kafka.apache.org/documentation/#brokerconfigs_min.insync.replicas)
            - The name of the param is the same at topic level.
            - This parameter is used to determine the minimum number of in-sync replicas (ISR) that must acknowledge a write for the write to be considered successful.
            - If the number of in-sync replicas falls below this value, the broker will stop accepting writes until the ISR count is restored.
            - This parameter is used to ensure that data is not lost if a broker fails ( Consistency ).
        - [num.io.threads](https://kafka.apache.org/documentation/#brokerconfigs_num.io.threads)
        - [num.network.threads](https://kafka.apache.org/documentation/#brokerconfigs_num.network.threads)
        - [num.recovery.threads.per.data.dir](https://kafka.apache.org/documentation/#brokerconfigs_num.recovery.threads.per.data.
        dir)
        - [num.replica.fetchers](https://kafka.apache.org/documentation/#brokerconfigs_num.replica.fetchers)
        - [offsets.retention.minutes](https://kafka.apache.org/documentation/#brokerconfigs_offsets.retention.minutes)
        - [unclean.leader.election.enable](https://kafka.apache.org/documentation/#brokerconfigs_unclean.leader.election.enable)
            - This parameter is used to determine whether a broker can be elected as a leader if it is not in sync with the ISR.
            - Setting `unclean.leader.election.enable` to true allows out-of-sync replicas to become leaders (known as unclean election), with the understanding that this may result in the loss of messages.
            - If set to false(default), the broker must be in sync with the ISR to be elected as a leader.
        - [zookeeper.session.timeout.ms](https://kafka.apache.org/documentation/#brokerconfigs_zookeeper.session.timeout.ms)
        - [broker.rack](https://kafka.apache.org/documentation/#brokerconfigs_broker.rack)
        - [default.replication.factor](https://kafka.apache.org/documentation/#brokerconfigs_default.replication.factor)
        - [num.partitions](https://kafka.apache.org/documentation/#brokerconfigs_num.partitions)
        - []()

#### Troubleshooting

- DumpLogSegment: Kafka brokers include the **DumpLogSegment** tool, which enables you to view a partition segment in the filesystem and examine its contents.
    - This tool is useful for debugging and troubleshooting issues with the log segments.
    - `bin/kafka-run-class.sh kafka.tools.DumpLogSegments`: This command will dump the contents of a log segment file to the console.


#### CPU, RAM, Network, Storage Considerations

#### OS

- [📺 40. Kafka Performance: OS (Operating System)](https://www.udemy.com/course/kafka-cluster-setup/)
- [OS](https://kafka.apache.org/documentation/#os)
- Use Unix systems, tested on Linux and Solaris.
- We recommend at least 100,000 allowed file descriptors for the broker processes as a starting point.
- Run Kafka and Zookeeper independently; avoid putting them on the same VM.

#### CPU

- [39. Kafka Performance: CPU](https://www.udemy.com/course/kafka-cluster-setup/)
- [Find all the CPU-related subjects in the docs](https://kafka.apache.org/documentation/)
- In some cases, the bottleneck is not CPU or disk but network bandwidth.
- SSL data in transit causes CPU load due to encrypting/decrypting the message payload.
- Avoid letting the Kafka Broker compress messages; instead, let the producers and consumers handle it to reduce CPU load on the broker side.
- Monitor Java GC closely to avoid long pauses.

#### RAM

- [📺 38. Kafka Performance: RAM](https://www.udemy.com/course/kafka-cluster-setup/)
- [Understanding Linux OS Flush Behavior](https://kafka.apache.org/documentation/#linuxflush)
- Understand the importance of using `pagecache`:
    - It improves Kafka performance by acting as a memory buffer before data is written to disk.
    - Java Heap Memory is used for the Kafka Java process.
    - The rest of the available free RAM is used for the `pagecache`.
- Recommended to have Kafka broker instances with a minimum of 8GB in production, preferably 16GB or 32GB.
- Disable SWAP for Kafka: set `vm.swappiness=0` or `vm.swappiness=1`.

##### Garbage Collector
- Tuning the Java garbage-collection options
- G1 is designed to automatically adjust to different workloads and provide consis‐
tent pause times for garbage collection over the lifetime of the application.
- It also
handles large heap sizes with ease by segmenting the heap into smaller zones and not
collecting over the entire heap in each pause.
- There are two configuration options for G1 used to adjust its performance:
    - `MaxGCPauseMillis`: This option specifies the preferred pause time for each garbage-collection cycle.
It is not a fixed maximum—G1 can and will exceed this time if it is required. This
value defaults to 200 milliseconds. This means that G1 will attempt to schedule
the frequency of GC cycles, as well as the number of zones that are collected in
each cycle, such that each cycle will take approximately 200ms.
    - `InitiatingHeapOccupancyPercent`: This option specifies the percentage of the total heap that may be in use before
G1 will start a collection cycle. The default value is 45. This means that G1 will
not start a collection cycle until after 45% of the heap is in use. This includes both
the new (Eden) and old zone usage in total.

#### Network

- [📺 37. Kafka Performance: Network](https://www.udemy.com/course/kafka-cluster-setup/)
- [Operationalizing ZooKeeper](https://kafka.apache.org/documentation/#zkops)
- Latency is critical in Kafka.
- Place Kafka Brokers and Zookeeper as close as possible, ideally within the same region (for cloud setups).
- Avoid placing all components in the same AZ or rack, as it increases the risk of failure, though it may reduce latency.
- Bandwidth is key in Kafka.
- The network can be a bottleneck.
- Ensure sufficient bandwidth to handle multiple connections and TCP requests.
- Guarantee high network performance.
- Monitor the network to identify when it becomes a bottleneck.
- Kernel is not tuned by default for large, high-speed data transfers by default.
- Parameters:
    - Send and receive buffer default size per socket are:
        - net.core.wmem_default and net.core.rmem_default: 131072, 128 KiB
    - Send and receive buffer maximum sizes are:
        - net.core.wmem_max and net.core.rmem_max: 2097152, 2 MiB
    - Send and receive buffer sizes for TCP sockets must be set separately:
        - net.ipv4.tcp_wmem and net.ipv4.tcp_rmem param‐meters. 
        An example setting for each of these parameters is “4096 65536
        2048000,” which is a 4 KiB minimum, 64 KiB default, and 2 MiB maximum buffer.
- Enabling TCP window scaling by setting net.ipv4.tcp_window_scaling to 1: will allow clients
    to transfer data more efficiently, and allow that data to be buffered on the broker side.
- Increasing the value of net.ipv4.tcp_max_syn_backlog above the default of 1024
    will allow a greater number of simultaneous connections to be accepted.
- Increasing
    the value of net.core.netdev_max_backlog to greater than the default of 1000 can
    assist with bursts of network traffic, specifically when using multigigabit network
    connection speeds

#### Storage Considerations

- [📺 36. Kafka Performance: I/O](https://www.udemy.com/course/kafka-cluster-setup/)
- [Disks and Filesystem](https://kafka.apache.org/documentation/#diskandfs)
- [Filesystem Selection](https://kafka.apache.org/documentation/#filesystems)
- Reads are done sequentially; use a disk type that matches this requirement.
- Format drives as XFS.
- If there is a read/write throughput bottleneck:
    - Mount multiple disks in parallel for Kafka.
    - Use the configuration: `log.dirs=/disk1/kafka-logs,/disk2/kafka-logs...`.
- Kafka performance remains constant regardless of the amount of data stored.
    - Expire data quickly (default: 1 week).
    - Monitor disk performance.
- Use XFS
- XFS also has better performance for Kafka’s workload without
requiring tuning beyond the automatic tuning performed by the filesystem.
- EXT4 -> XFS (Better option)
- More efficient when batching disk writes
- Better overall I/O throughput
- Set noatime mount option for the mount point. creation time (ctime), last modified time (mtime), and last access time (atime)

### Number of Nodes
- Determining the number of nodes for a specific cluster is influenced by several factors:
    - How much disk capacity is required for retaining messages?
    - How much storage is available on a single broker?
        e.g. If the cluster is required to retain 10 TB of data and a single broker can store 2 TB, then the minimum cluster size is five brokers. 
        In addition, using replication will increase the storage requirements by at least
        100%, depending on the replication factor chosen. This means that
        this same cluster, configured with replication, now needs to contain at least 10 brokers.
    - Traffic request. 
        - Can that cluster handle all the requests?
        - What is the capacity of the network interfaces?
        - How many consumer do we have?
            - Is the replication needed?

### Rack Awareness
- The best practice is to have each Kafka broker in a cluster installed in a different rack,
or at the very least not share single points of failure for infrastructure services such as
power and network.
- Partition(replica) allocation rack awareness: If the brokers have rack information (available in Kafka release 0.10.0 and higher), Kafka attempts to assign the replicas for each partition to different racks whenever possible. This helps ensure that an event causing downtime for an entire rack does not lead to complete unavailability of the partitions.


### Kafka Connect
🚧

#### Scalability and High Availability

- [📺 42. Running Kafka in Production on AWS](https://www.udemy.com/course/kafka-cluster-setup/)
    - Separate intances between AZs
    - Use `st1` EBS volumes for best price / performance ratio
    - Mount multiple EBS volumes to the same broker if we need to scale
    - Use `r4.xlarge` or `m4.2xlarge` EBS optimize instances
    - Use DNS names or fixed IP's for the brokers to don't impact the clients.

### Business Continuity / DR
Mirror maker

### Data Retention

- Administrator configures a retention period for each topic, which specifies either the duration to store messages before they are deleted or the amount of data to retain before older messages are purged.
- Each partition is split into segments. By default, each segment contains either 1 GB of data or data from one week, whichever is smaller.
- When a Kafka broker is writing to a partition, if the segment limit is reached, it closes the current file and begins writing to a new one.


