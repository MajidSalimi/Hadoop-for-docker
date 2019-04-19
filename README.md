## Deploy Hadoop in Cloud environment

### Step1: install Docker Toolbox

first, You should donwload docker toolbox from this link:

https://download.docker.com/win/stable/DockerToolbox.exe

Now install Docker. After of the installation, open the "Docker Quickstart Terminal" and test your docker to be installed successfully.

Before continuing, you should connect your proxy!
Open doker toolbox and
Run ```docker --version``` to ensure that you have a supported version of Docker.
```
$docker --version

Docker version 17.12.0-ce, build c97c6d6
```

Run ```docker info``` (or docker version without ```--```) to view even more details about your Docker installation.
```
$docker info

Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.12.0-ce
Storage Driver: overlay2
...
```
#### Test Docker installation:

1. Test that your installation works by running the simple Docker image, hello-world:
```
$docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:ca0eeb6fb05351dfc8759c20733c91def84cb8007aa89a5bf606bc8b315b9fc7
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```
2. List the hello-world image that was downloaded to your machine:
```
$docker image ls
```
3. List the hello-world container (spawned by the image) which exits after displaying its message. If it were still running, you would not need the --all option:
```
$docker container ls --all

CONTAINER ID     IMAGE           COMMAND      CREATED            STATUS
54f4984ed6a8     hello-world     "/hello"     20 seconds ago     Exited (0) 19 seconds ago
```

### Step2: Download Docker files of Hadoop
Download files from the link : https://github.com/MajidSalimi/Hadoop-for-docker and extract them in a specified folder.

### Step3: Setup Swarm 
To setup a cluster which has the ability of
Service discovery
Load balance
Retry on failure
Fast deploy
```
$docker swarm init --advertise-addr 127.0.0.1 
```
### Step4: Create an overlay network
```
$docker network create --driver overlay swarm-net 
```
### Step5: Build docker image
Go to your directory of Docker files (with ```cd``` command) and run this command to Build docker image:
```
$docker build --tag newnius/hadoop:3.7 .
```
It will take a few time to complete.

### Step6: Start Hadoop Cluster
```
$docker service create \
--name hadoop-master \
--network swarm-net \
--hostname hadoop-master \
--replicas 1 \
--endpoint-mode dnsrr \
newnius/hadoop:3.7 
```

```
$docker service create \
--name hadoop-slave1 \
--network swarm-net \
--hostname hadoop-slave1 \
--replicas 1 \
--endpoint-mode dnsrr \
newnius/hadoop:3.7 
```

```
$docker service create \
--name hadoop-slave2 \
--network swarm-net \
--hostname hadoop-slave2 \
--replicas 1 \
--endpoint-mode dnsrr \
newnius/hadoop:3.7 
```

```
$docker service create \
--name hadoop-slave3 \
--network swarm-net \
--hostname hadoop-slave3 \
--replicas 1 \
--endpoint-mode dnsrr \
newnius/hadoop:3.7 
```

##### check your cluster
now you can check your cluster with:

```
$docker service ls
```

### Step7: start a proxy to access Hadoop web UI
To monitor Hadoop yarn on:
http://hadoop-master:8088
you should use this command:

```
$docker service create \
--replicas 1 \
--name proxy_docker \
--network swarm-net \
-p 7001:7001 \
newnius/docker-proxy 
```

### Step8: Enter containers
You can list your containers with:

```$docker ps```

and enter the cntainer with the ContainerId that you should replace it in the following command:

```
$docker exec -it ContainerId bash 
```

### Step9: format namenode

In the first time we run Hadoop cluster, it is required to format HDFS in namenode.

```
# stop all Hadoop processes

sbin/stop-yarn.sh

sbin/stop-dfs.sh

stop-all.sh

# remove old files of hdfs in host filesystem in all nodes

rm -rf /tmp

# format namenode

bin/hadoop namenode -format

hadoop namenode -format

# start yarn and dfs nodes

sbin/start-dfs.sh

sbin/start-yarn.sh 

start-all.sh
```

### Step10: Run a test

prepare input files to hdfs:///user/root/input
```
bin/hdfs dfs -mkdir -p /user/root/input

bin/hdfs dfs -put etc/hadoop/* /user/root/input 
```

Run WordCount:

```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.4.jar wordcount input output 

```

Now you can check the web-UI in the: hadoop-master:8088 in your browser.



