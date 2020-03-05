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

# 3. Install Docker.
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
# 4. Docker post-installation Steps
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

# 5. Cassandra Docker Cluster
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
## 5.1 Download Locally Cassandra Docker Image
- Run <kbd>$ docker pull cassandra</kbd>
- You can specify a version <kbd>$ docker pull cassandra:3.11.6</kbd>
## 5.2 Creating a Docker Cassandra Cluster For Climatos With Persistecy
- Execute <kbd>$ docker run --name cassandra_base -d cassandra:3.11.6</kbd>
- This will create a container with name cassandra_base of cassandra version 3.11.6
- Now we need to copy configuration files and database files from this container
- To start the container execute <kbd>$ docker start cassandra_base</kbd>
- Change the owner of the /opt folder <kbd>$ sudo chown climatos /opt</kbd>
  - If you have any permission problem along the way check the file with <kbd>$ ls -la</kbd> and if the folder not belong to the user try the ***chown*** command. 
- Now copy the file <kbd>$ docker cp cassandra_base:/opt/cassandra /opt/cassandra</kbd>
- Go to /opt/cassandra <kbd>$ cd /opt/cassandra</kbd>
- Exclude conf and data folders <kbd>$ rm -R conf data</kbd>
- Make another copy from container <kbs>$ docker cp cassandra_base:/opt/cassandra/conf/. /opt/cassandra/conf</kbd>
  - for a unknow reason wen you make the first copy you get a single file inside /opt/cassandra/conf and it is useless, that's why we make a second copy
- Now create a container that will act as a climatos cassandra node <kbd>$ docker run --name climatos --privileged -p 7000:7000 -e CASSANDRA_BROADCAST_ADDRESS=your_currente_ip_address -e CASSANDRA_CLUSTER_NAME=climatos -v /opt/cassandra/conf:/opt/cassandra/conf -v /opt/cassandra/data/data:/opt/cassandra/data/data cassandra:3.11.6</kbd>
  - your_currente_ip_address means: the ip address of your current machine
  - CASSANDRA_CLUSTER_NAME have to be equal in all nodes
  - -v path/in/local/machine:/path/in/container
  - To execute this commands in other machines <kbd>user@machine_n $ docker run --name climatos --privileged -p 7000:7000 -e CASSANDRA_BROADCAST_ADDRESS=your_currente_ip_address -e CASSANDRA_CLUSTER_NAME=climatos -e CASSANDRA_SEEDS=10.10.0.1,10.10.10.2 -v /opt/cassandra/conf:/opt/cassandra/conf -v /opt/cassandra/data/data:/opt/cassandra/data/data cassandra:3.11.6</kbd>
    - We are assuming: Machine1: 10.10.10.1 Machine2: 10.10.10.2 that's why "seeds" have two address.
## 5.3 Cassandra.yaml little adjustments
- In all nodes open <kbd>/opt/cassandra/conf/cassandra.yaml</kbd> with nano or vim. Search for "authenticator: " and change the value "AllowAllAuthenticator" to "PasswordAuthenticator"
  - You can change seeds too (search for "seeds: ")
# 6 Useful Commands
## 6.4 Coping folders to a container
- To copy from a container <kbd>$ docker cp container_name:/path/to/file /path/to/host</kbd>

## 6.5 Deleting a container
- <kbd>$ docker container rm container_name</kbd>

## 6.6 Stoping a container
- <kbd>$ docker stop container_name</kbd>

## 6.7 Using bash in a container
- <kbd>$ docker exec -ti container_name bash</kbd>
  
## 6.1 Creating Superuser Accounts in cqlsh
- Open the bash of the container
- Execute <kbd>$ cqlsh -u cassandra -p cassandra</kbd>
- <kbd>cassandra@cqlsh> CREATE ROLE new_root_user WITH SUPERUSER = true AND LOGIN = true AND PASSWORD = 'password';</kbd>
- <kbd>cassandra@cqlsh> EXIT;</kbd>
- <kbd>$ cqlsh -u new_root_user</kbd>
- Enter the password
- <kbd>new_root_user@cqlsh> LIST ROLES;</kbd>
- The result is something like this:
```

 role                | super | login | options
---------------------+-------+-------+---------
           root_user |  True |  True |        {}
           cassandra |  True |  True |        {}
(2 rows)
```
- To change cassandra user permission, execute <kbd>new_root_user@cqlsh> ALTER ROLE cassandra WITH SUPERUSER = false AND LOGIN = false AND password='new_secret_pw';</kbd>
- To drop cassandra account: <kbd>new_root_user@cqlsh> DROP ROLE cassandra;</kbd>
- Exit from cqlsh <kbd>new_root_user@cqlsh EXIT;</kbd>
## 6.2 Droping node from cluster ring.
- nodetool decommission
## 6.3 Renaming the Cluster
- <kbd>$ cqls -u user_name</kbd>
- <kbd>cqlsh> update system.local set cluster_name = 'new_cluster_name' where key='local';</kbd>
- <kbd>cqls> exit</kbd>
- Change the field "cluster_name" in /etc/cassandra/cassandra.yaml to the 'new_cluster_name'
- <kbd>$ nodetool flush -- system</kbd>

# 7 References
- https://hub.docker.com/_/cassandra
- https://docs.docker.com/install/linux/docker-ce/centos/
- 