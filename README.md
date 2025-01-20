
# Hadoop Docker



## Quick Start

### Install Docker (Linux/MacOS/Windows)

Follow these steps to install Docker Desktop:

1. Download Docker Desktop:
  -- Go to [Docker.com](https://www.docker.com/) and choose the right version for your computer.


2. Run the installer and follow the installation wizard
  - Accept the terms and conditions
  - Choose default settings (recommended)
  - Click Install

3. Start Docker Desktop after installation

4. Verify installation by opening a terminal and running:

```
$ docker --version
$ docker-compose --version
```




### Clone the Repository

You have two options to get the repository:

1. Using Git (Recommended):
```bash
git clone https://github.com/Abuhamad/hadoop.git
cd hadoop
```

2. Manual Download:
  - Visit [hadoop repository](https://github.com/Abuhamad/hadoop)
  - Click the green "Code" button
  - Select "Download ZIP"
  - Extract the ZIP file to your preferred location


### Deploy a Hadoop cluster

This guide explains how to deploy a multi-node Hadoop cluster using Docker Compose. The deployment includes:

#### Prerequisites
- Docker and Docker Compose installed
- At least 8GB RAM available
- 20GB free disk space

#### Cluster Architecture
The deployment creates a complete Hadoop ecosystem with:
- HDFS (Distributed File System)
  - 1 NameNode: Manages filesystem namespace
  - 3 DataNodes: Stores actual data blocks
- YARN (Resource Management)
  - 1 ResourceManager: Manages resources and applications
  - 1 NodeManager: Handles containers and node resources
- 1 HistoryServer: Tracks completed applications

#### Key Web UIs (after deployment)
- NameNode: http://DockerIP:9870
- ResourceManager: http://DockerIP:8088
- HistoryServer: http://DockerIP:19888

#### Notes
- Initial startup takes 1-2 minutes for all services to be ready
- Use `docker-compose logs -f` to monitor startup progress
- The cluster is configured for development/testing purposes
- Default configuration uses 3 DataNodes for redundancy
- All data is ephemeral and will be lost when containers are removed

#### Troubleshooting
- If containers fail to start, check Docker resource limits
- Ensure no port conflicts with existing services
- For persistent storage, modify volume mappings in docker-compose.yml

### Deploy a Hadoop cluster

1. Navigate to the hadoop directory:
```bash
cd hadoop
```

2. Start the Hadoop cluster using docker-compose:
```bash
docker-compose up -d
```

This will:
- Pull required Docker images
- Create containers defined in docker-compose.yml:
  - 1 NameNode (HDFS master)
  - 3 DataNodes (HDFS workers) 
  - 1 ResourceManager (YARN)
  - 1 NodeManager (YARN)
  - 1 HistoryServer

3. Wait for containers to start (1-2 minutes)

4. Verify cluster is running:
```bash
docker ps
```
<!-- 
Also, you can go to http://DockerIP:9870/ from your browser to see the namenode status. (Docker IP from by running:"docker-machine config default") -->

You should see containers with status "Up":
- namenode
- datanode-1/2/3  
- resourcemanager
- nodemanager
- historyserver

The -d flag runs containers in detached mode (background). Remove it to see logs in realtime.







### Running MapReduce Jobs

To run other MapReduce jobs, follow this pattern:

1. Copy your JAR file to the namenode:
```bash
docker cp ./your-mapreduce.jar namenode:/your-mapreduce.jar
```

2. Access the namenode container:
```bash
docker exec -it namenode bash
```

3. Run your MapReduce job:
```bash
hadoop jar your-mapreduce.jar MainClass input_path output_path
```

4. View results:
```bash
hadoop fs -cat output_path/part-r-00000
```

Common MapReduce examples included in Hadoop:
- wordcount: Counts word occurrences
- pi: Calculates pi using map-reduce
- terasort: Sorts data at terabyte scale




### Shut Down the Hadoop Cluster

When you're done using the cluster, shut it down to free up system resources:

```bash
docker-compose down
```

This command will:
- Stop all running containers
- Remove the containers
- Remove default networks

Note: All data in the HDFS will be lost when containers are removed.



### Restart the Hadoop Cluster

To restart the cluster with the same container configuration:

```bash
docker-compose up -d
```

This command will:
- Create new containers using the same configuration
- Start all services defined in docker-compose.yml
- Restore the cluster to its initial state

Note: Any previous HDFS data will not be restored.



### Delete All Docker Containers

To remove all containers and associated resources:

```bash
# Stop and remove all containers
docker-compose down --volumes --rmi all

# Verify all containers are removed
docker ps -a
```

This command will:
- Stop all running containers
- Remove all stopped containers
- Remove all networks
- Remove all volumes
- Remove all images used by any service