
# Hadoop Docker



## Quick Start

### Install Docker (Linux/MacOS/Windows)

Follow these steps to install Docker Desktop:

1. Download Docker Desktop:
  -  Go to [Docker.com](https://www.docker.com/) and choose the right version for your computer.


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




### Stop and Start the Hadoop Cluster

To temporarily stop the cluster without removing containers:

```bash
docker-compose stop
```

This command will:
- Stop all running containers
- Preserve container data and configuration

To restart the stopped containers:

```bash
docker-compose start
```

This command will:
- Start all previously stopped containers
- Maintain existing data and configuration


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


---

## MapReduce Jobs

MapReduce is a programming model and framework for processing large datasets in parallel across a distributed cluster of computers. It was introduced by Google in a seminal paper in 2004 and later popularized by Apache Hadoop, which provides an open-source implementation of the MapReduce model. MapReduce is designed to handle big data workloads by dividing tasks into smaller, manageable chunks that can be processed in parallel.

### MapReduce in Hadoop
In Hadoop, MapReduce jobs are executed using YARN (Yet Another Resource Negotiator), which manages resources and schedules tasks across the cluster. The key components are:

1. **ResourceManager:** Manages resources and schedules jobs.
2. **NodeManager:** Runs tasks on individual nodes.
3. **ApplicationMaster:** Manages the lifecycle of a MapReduce job.

### Basic Steps to Run MapReduce Jobs

1. Verify cluster status:
```bash
docker ps
```
Look for running containers: namenode, resourcemanager, datanodes, etc.

2. Connect to namenode:
```bash
docker exec -it namenode bash
```

3. Compile and package code:
```bash
# Inside namenode container
mkdir wordcount
cd wordcount
# Write and Copy your .java files here (see next steps)
```

4. Verify YARN status:
```bash
# Show node list
yarn node -list
# OR show application list
yarn application -list
```

If these commands don't show any nodes or return errors:

1. Check if YARN services are running:
```bash
docker ps | grep resourcemanager
docker ps | grep nodemanager
```

2. If services are down, restart them:
```bash
docker-compose restart resourcemanager nodemanager
```
If they the containers are running but still no results, mannually start them:
```bash
yarn --daemon start resourcemanager

#if the nodemanager is not working
yarn --daemon start nodemanager
```


3. Wait 30 seconds for services to initialize, then try again:
```bash
yarn node -list
```

5. Run MapReduce job:
```bash
hadoop jar wordcount.jar WordCount /input /output
```


### Compile and Run Java MapReduce Code

#### 1. Write Your MapReduce Code
Save your MapReduce program in a Java file, e.g., `WordCount.java`.

#### 2. Compile the Java Code
Use `javac` to compile your Java program. Include the Hadoop classpath.

```bash
javac -classpath `$HADOOP_HOME/bin/hadoop classpath` -d . WordCount.java
```
- `-classpath`: Specifies the Hadoop libraries.
- `-d .`: Outputs the compiled `.class` files to the current directory.

### 3. Package the Compiled Classes into a JAR
Package the compiled `.class` files into a JAR file.

```bash
jar -cvf WordCount.jar -C . .
```
- `-cvf`: Creates a JAR file (WordCount.jar).
- `-C . .`: Includes all `.class` files in the current directory.

### 4. Run the MapReduce Job
Submit the JAR file to Hadoop using the hadoop jar command.

```bash
hadoop jar WordCount.jar WordCount /input /output
```
Parameters:
- `WordCount.jar`: The JAR file containing your MapReduce program.
- `WordCount`: The main class name.
- `/input`: The input directory on HDFS.
- `/output`: The output directory on HDFS (must not exist before running).

### 5. Check the Output
View the output of your MapReduce job.

```bash
hadoop fs -cat /output/part-r-00000
```


### 