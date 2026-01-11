# Hadoop Docker Setup Scripts

This project provides Docker-based setup scripts for creating Hadoop clusters with YARN and MapReduce support. It includes both single-node and multi-node cluster configurations.

## Overview

The project contains two complete shell scripts for setting up Hadoop clusters using Docker:

1. **hadoop-single-node.txt** - Single-node Hadoop cluster (pseudo-distributed mode)
2. **hadoop-multi-node.txt** - Multi-node Hadoop cluster (1 NameNode + 2 DataNodes)

Both scripts are self-contained and include:
- Dockerfile definitions
- Complete Hadoop configuration
- Service startup scripts
- MapReduce example executions
- UTF-8 encoded descriptions (Polish and English)
- Commented terminal output

## Prerequisites

- **Docker** - Docker Engine installed and running
- **Linux/macOS** - Scripts are written for Unix-like systems
- **Bash shell** - Scripts require bash shell
- **Internet connection** - For downloading Hadoop distribution during build
- **Sufficient resources** - At least 4GB RAM recommended for multi-node setup

## Hadoop Version

- **Hadoop**: 3.3.6
- **Java**: OpenJDK 11
- **Base Image**: Ubuntu 20.04

## Quick Start

### Single-Node Cluster

Run the single-node setup script:

```bash
bash hadoop-single-node.txt
```

This will:
1. Build a Docker image with Hadoop 3.3.6
2. Start a container with all Hadoop services
3. Configure Hadoop in pseudo-distributed mode
4. Start HDFS and YARN services
5. Run MapReduce examples (Pi calculation and wordcount)

### Multi-Node Cluster

Run the multi-node setup script:

```bash
bash hadoop-multi-node.txt
```

This will:
1. Build a Docker image with Hadoop 3.3.6
2. Create a Docker network for cluster communication
3. Start 3 containers (1 NameNode + 2 DataNodes)
4. Configure SSH for passwordless access
5. Configure Hadoop cluster files
6. Start all Hadoop services
7. Run MapReduce examples demonstrating cluster functionality

## Architecture

### Single-Node Setup

- **1 Container**: Runs all Hadoop services
  - NameNode
  - DataNode
  - SecondaryNameNode
  - ResourceManager
  - NodeManager

### Multi-Node Setup

- **3 Containers**:
  - **NameNode** (172.25.0.10): NameNode, ResourceManager, SecondaryNameNode, JobHistoryServer
  - **DataNode1** (172.25.0.11): DataNode, NodeManager
  - **DataNode2** (172.25.0.12): DataNode, NodeManager

- **Network**: Docker bridge network (hadoop-net) with subnet 172.25.0.0/16

## Web Interfaces

After running the scripts, access the following web UIs:

### Single-Node

- **NameNode Web UI**: http://localhost:9870
- **YARN ResourceManager**: http://localhost:8088

### Multi-Node

- **NameNode Web UI**: http://localhost:9870
- **YARN ResourceManager**: http://localhost:8088
- **Job History Server**: http://localhost:19888

## File Structure

```
.
├── hadoop-single-node.txt    # Single-node cluster setup script
├── hadoop-multi-node.txt     # Multi-node cluster setup script
└── README.md                 # This file
```

## Script Features

### Common Features

Both scripts include:
- ✅ Automatic cleanup of existing containers/images
- ✅ Complete Hadoop configuration (core-site.xml, hdfs-site.xml, yarn-site.xml, mapred-site.xml)
- ✅ YARN configuration for MapReduce execution
- ✅ SSH setup for passwordless access
- ✅ MapReduce examples (Pi calculation and wordcount)
- ✅ UTF-8 encoded step descriptions (Polish and English)
- ✅ Commented terminal output

### Single-Node Specific

- Pseudo-distributed mode (all services on one node)
- Replication factor: 1
- All services on localhost

### Multi-Node Specific

- Fully distributed cluster
- Replication factor: 2
- Docker network for inter-container communication
- SSH key distribution for cluster operations
- Workers file configuration for DataNodes

## Configuration Details

### Hadoop Configuration Files

- **core-site.xml**: File system default configuration
- **hdfs-site.xml**: HDFS-specific settings (replication, data directories)
- **yarn-site.xml**: YARN resource management configuration
- **mapred-site.xml**: MapReduce framework configuration with YARN
- **workers**: List of DataNode hostnames (multi-node only)

### Environment Variables

The scripts configure user environment variables in `hadoop-env.sh`:
- `HDFS_NAMENODE_USER=root`
- `HDFS_DATANODE_USER=root`
- `HDFS_SECONDARYNAMENODE_USER=root`
- `YARN_RESOURCEMANAGER_USER=root`
- `YARN_NODEMANAGER_USER=root`

### Ports Exposed

**Single-Node:**
- 9870: NameNode Web UI
- 8088: YARN ResourceManager
- 9000: HDFS NameNode RPC

**Multi-Node:**
- 9870: NameNode Web UI
- 8088: YARN ResourceManager
- 19888: MapReduce Job History Server

## Usage Examples

### Running MapReduce Jobs

After the setup is complete, you can run MapReduce jobs using:

```bash
# Single-node
docker exec hadoop-single hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar <example> <args>

# Multi-node
docker exec namenode hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.3.6.jar <example> <args>
```

Available examples include:
- `wordcount`: Count word frequencies
- `pi`: Estimate Pi using Monte Carlo method
- `grep`: Search text patterns
- `sort`: Sort data
- And more...

### HDFS Commands

```bash
# Single-node
docker exec hadoop-single hadoop fs -ls /
docker exec hadoop-single hadoop fs -mkdir /mydir
docker exec hadoop-single hadoop fs -put localfile /hdfs/path

# Multi-node
docker exec namenode hadoop fs -ls /
docker exec namenode hadoop fs -mkdir /mydir
docker exec namenode hadoop fs -put localfile /hdfs/path
```

### Checking Services

```bash
# Single-node
docker exec hadoop-single jps

# Multi-node
docker exec namenode jps
docker exec datanode1 jps
docker exec datanode2 jps
```

## Cleanup

### Stop Containers

**Single-Node:**
```bash
docker stop hadoop-single
docker rm hadoop-single
```

**Multi-Node:**
```bash
docker stop namenode datanode1 datanode2
docker rm namenode datanode1 datanode2
docker network rm hadoop-net
```

### Remove Images

```bash
# Single-node
docker rmi hadoop-single-node

# Multi-node
docker rmi hadoop-multi-node
```

## Troubleshooting

### Containers Exit Immediately

If containers exit immediately after starting, check:
- Docker daemon is running
- Sufficient disk space
- Check logs: `docker logs <container-name>`

### Services Not Starting

If Hadoop services fail to start:
- Check SSH service is running: `docker exec <container> service ssh status`
- Verify Java installation: `docker exec <container> java -version`
- Check Hadoop logs: `docker exec <container> cat /usr/local/hadoop/logs/*.log`

### MapReduce Jobs Fail

If MapReduce jobs fail:
- Verify YARN services are running (ResourceManager, NodeManager)
- Check HDFS is accessible
- Ensure mapred-site.xml configuration is correct
- Check YARN logs for errors

### Port Conflicts

If ports are already in use:
- Stop conflicting services
- Or modify port mappings in the docker run commands

## Technical Notes

- Scripts use temporary directories for Docker build context
- Containers are kept running with `tail -f /dev/null`
- SSH is configured for passwordless access within the cluster
- All Hadoop configuration files are created programmatically
- User environment variables are set to allow running as root (appropriate for Docker containers)

## License

This project is provided as-is for educational purposes.

## Author

Created for educational purposes as part of Cloud Computing coursework.

## References

- [Apache Hadoop Documentation](https://hadoop.apache.org/docs/current/)
- [Hadoop Single Node Setup](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)
- [Hadoop Cluster Setup](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)
