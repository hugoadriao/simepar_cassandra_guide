# 1. Login into server
## 1.1 SSH connection
- <kbd>$ root@SERVER_IP_ADDRESS</kbd>  
- Type "yes" after the warning
- Type the password
If everything is right, you will be logged in as root

# 2. Root
## 2.1 About root
The root is the administrative user in Linux that has very broad privileges. Because of the heightened privileges, you are discouraged from using it. This is because part of the power inherent with the root account is the ability to make very destructive changes, even by accident.
## 2.2 Create a New User
Once you are logged in as ***root***, we're prepared to add the new user account that we will use to log in from now on.
- <kbd>\# adduser username</kbd>  
- <kbd>\# passwd username</kbd>  
Enter a password, and repeat it to verify it.
## 2.3 Root Privileges
We have a new user account with regular privileges. However, we may sometimes need to do administrative tasks.  
  
To avoid having to log out of "normal user" and log back in as root, we can set up a "super user" privileges for our normal account. This will allow our normal user to run commands with administrative privileges by putting the word **sudo** before each command.  

By default, on CentOS 7, users who belong to the "wheel" group are allowed to use the **sudo** command.  

As **root**, run this command to add your new user to the *wheel* group.  
- <kbd>\# gpasswd -a username wheel</kbd>  
Now the user can run commands with super user privileges.  

## 2.4 Changing To New User.
To change to the new user, run this command.
- <kbd>\# su - username</kbd>  

## 3. Install Docker.
- Unistall old Docker version
```
  $ sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-engine
```
- Install some useful packages
```
  $ sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
```
-  Execute the following command to set up the stabe repository
```
    $ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
- Install the latest version of Docker Engine - Community and containerd - with the following command
<kbd>
  $ sudo yum install docker-ce docker-ce-cli containerd.io
</kbd>
## 4. Docker post-installation Steps
- Start Docker
<kbd>
sudo systemctl start docker
</kbd>
- Verify that Docker is installed correctly
<kbd>
$ sudo docker run hello-world
</kbd>
- The result should be, something like this:
```
To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
- Enable Docker to autostart <kbd>sudo systemctl enable docker</kbd>
- Add your user to the docker group to get rid of the need of typing ***sudo*** each time you run docker command <kbd>$ sudo usermod -aG docker $USER</kbd>

## 5. Cassandra Docker Cluster
Let's assume the following scenario:  
- Machine1:
  - IP: 10.10.10.1
  - Docker name: climatos1
- Machine2:
  - IP: 10.10.10.2
  - Docker name: climatos2
- Machine3:
  - IP: 10.10.10.3
  - Docker name: climatos3
- All in the same datacenter
### 5.1 Download Locally Cassandra Docker Image
- Run <kbd>$ docker pull cassandra</kbd>
- You can specify a version <kbd>$ docker pull cassandra:3.11.6</kbd>
### 5.1 Creating a Standard Cassandra Cluster
- <kbd>$ docker run --name docker_name -v /my/own/datadir:/var/lib/cassandra -d -e CASSANDRA_BROADCAST_ADDRESS=machine_ip_address -p 7000:7000 cassandra:tag</kbd>
where:
  - --name: You define an name to the image
  - CASSANDRA_BROADCAST_ADDRESS: it has to be the machine ip address
  - -p: Is an port to access docker
  - ":tag": Is where you define cassandra version.
- Create a folder that will save the Cassandra information <kbd>$ sudo mkdir /var/lib/cassandra</kbd>
- Using Machine1 as exemple, the command should be: <kbd>$ docker run --name climatos1 -v /var/lib/cassandra:/var/lib/cassandra -d -e CASSANDRA_BROADCAST_ADDRESS=10.10.10.1 -p 7000:7000 cassandra:3.11.6</kbd>
- To create Cassandra replication nodes:
  - Machine2: <kbd>$ docker run --name climatos2 -v /var/lib/cassandra:/var/lib/cassandra -d -e CASSANDRA_BROADCAST_ADDRESS=10.10.10.2 -p 7000:7000 -e CASSANDRA_SEEDS=10.10.10.1 cassandra:3.11.6</kbd>
  - Machine3: <kbd>$ docker run --name climatos3 -v /var/lib/cassandra:/var/lib/cassandra -d -e CASSANDRA_BROADCAST_ADDRESS=10.10.10.3 -p 7000:7000 -e CASSANDRA_SEEDS=10.10.10.1 cassandra:3.11.6</kbd>
### 5.2 Starting Cassandra Docker Cluster
- To start a docker <kbd>$ docker start docker_name</kbd>
- Using Machine1 as example: <kbd>$ docker start climatos1</kbd>
## x. Useful Commands to Cassandra
### x.x Droping node from cluster ring.
- nodetool decommission
### x.x Renaming the Cluster
- <kbd>$ cqls</kbd>
- <kbd>cqlsh> update system.local set cluster_name = 'new_cluster_name' where key='local'</kbd>
- <kbd>cqls> exit</kbd>
- Change the field "cluster_name" in /etc/cassandra/cassandra.yaml to the 'new_cluster_name'
- <kbd>$ nodetool flush -- system</kbd>

# X. References
- https://hub.docker.com/_/cassandra
