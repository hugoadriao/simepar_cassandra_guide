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

# 3. Install Java 8.
## 3.1 Install wget.
To install wget run this command.
- <kbd>$ sudo yum install wget</kbd>
## 3.2 Install Java 8.
To install OpenJDK 8 JDK run this command.
- <kbd>$ cd ~</kbd>

To download Java execute:
- <kbd>$ wget --no-cookies --no-check-certificate --header "Cookie:oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm"</kbd>

To install Java execute the following command.
- <kbd>yum -y localinstall jdk-8u131-linux-x64.rpm</kbd>
## 3.3 Checking installation.
You can now check the ***Java*** version using the following command.
- <kbd>$ java -version  </kbd>  
  
You alson need to check if **JAVA_HOME** environment variable is set. Run this command.
- <kbd>$ echo $JAVA_HOME</kbd>  

If you get a null or blank output, you'll need manually set the **JAVA_HOME** variable.  
### 3.3.1 Manually setting JAVA_HOME variable
<blockquote>
The next step can be made with vim, but if you don't have familiarity, you can use **nano**.  

- <kbd>$ yum install nano</kbd>
</blockquote>

- <kbd>$ nano ~/.bash_profile</kbd>

Now add the copied path at the end of file and edit it to looks like the following

<blockquote>
    export JAVA_HOME=/usr/java/jdk1.8.0_131/
</blockquote>
<blockquote>
    export JRE_HOME=/usr/java/jdk1.8.0_131/jre 
</blockquote>

The command <kbd>$ echo $JAVA_HOME</kbd> should return something like:  
<kbd>/usr/java/jdk1.8.0_131/</kbd>

## 4. Install Cassandra
For this part we will assume that we have this scenario:
- Machine 1:
  - IP: 192.168.122.100
  - OS: CentOS 7
- Machine 2:
  - IP: 192.168.122.101
  - OS: CentOS 7
- Machine 3:
  - IP: 192.168.122.102
  - OS: CentOS 7
### 4.x Cassandra Configuration
- Open the file <kbd>$ sudo nano /etc/cassandra/conf/cassandra-env.sh</kbd>
- Change the line <kbd> JVM_OPTS="\$JVM_OPTS -Djava.rmi.server.hostname=\<localhost></kbd> to <kbd>JVM_OPTS="$JVM_OPTS -Djava.rmi.server.hostname=192.168.122.10X"</kbd>  
**change the "*X*" to correspond with the IP**
- Open the file <kbd>$ sudo nano /etc/rc.d/init.d/cassandra</kbd>
- Change the following lines:
 ```
  # Cassandra startup  
    echo -n "Starting Cassandra: "  
    [ -d `dirname "$pid_file"` ] || \  
        install -m 755 -o $CASSANDRA_OWNR -g $CASSANDRA_OWNR -d `dirname $pid_file`  
    su $CASSANDRA_OWNR -c "$CASSANDRA_PROG -p $pid_file" > $log_file 2>&1 
    retval=$?  
    [ $retval -eq 0 ] && touch $lock_file  
    echo "OK"  
    ;;  
```
```
# Cassandra startup
    echo -n "Starting Cassandra: "
    [ -d `dirname "$pid_file"` ] || \
        install -m 755 -o $CASSANDRA_OWNR -g $CASSANDRA_OWNR -d `dirname $pid_file`
    # su $CASSANDRA_OWNR -c "$CASSANDRA_PROG -p $pid_file" > $log_file 2>&1
    runuser -u $CASSANDRA_OWNR -- $CASSANDRA_PROG -p $pid_file > $log_file 2>&1
    chown root.root $pid_file
    retval=$?
    [ $retval -eq 0 ] && touch $lock_file
    echo "OK"
    ;;
```
- In the same file <kbd>/etc/rc.d/init.d/cassandra</kbd>
- Change: 
```
# Cassandra shutdown
        echo -n "Shutdown Cassandra: "
        su $CASSANDRA_OWNR -c "kill `cat $pid_file`"
        retval=$?
        [ $retval -eq 0 ] && rm -f $lock_file
        for t in `seq 40`; do
            status -p $pid_file cassandra > /dev/null 2>&1
            retval=$?
            if [ $retval -eq 3 ]; then
                echo "OK"
                exit 0
            else
                sleep 0.5
            fi;
        done
```
```
# Cassandra shutdown
        echo -n "Shutdown Cassandra: "
        # su $CASSANDRA_OWNR -c "kill `cat $pid_file`"
        runuser -u $CASSANDRA_OWNR -- kill `cat $pid_file`
        retval=$?
        [ $retval -eq 0 ] && rm -f $lock_file
        for t in `seq 40`; do
            status -p $pid_file cassandra > /dev/null 2>&1
            retval=$?
            if [ $retval -eq 3 ]; then
                echo "OK"
                exit 0
            else
                sleep 0.5
            fi;
        done
```
- Execute <kbd>$ systemctl daemon-reload</kbd>
- Start Cassandra <kbd>$ service cassandra start</kbd>
- <kbd>$ cqls</kbd>
- <kbd>cqlsh> update system.local set cluster_name = 'new_cluster_name' where key='local'</kbd>
- <kbd>cqls> exit</kbd>
- Open the file <kbd>$ sudo nano /etc/cassandra/conf/cassandra.yaml</kbd>
- The next x steps is in the same file
    - Search for 'cluster_name: ' and change it's value to the name of your's cassandra cluster (this is equal to all nodes)
    - Search for 'seeds' and insert the ip address of two cassandra nodes (more than this isn't recommended), for exemple, <kbd>- seeds: "192.168.122.100, 192.168.122.101"</kbd>
    - Search for 'listen_address: ' and change it's value to correspond with IP ADDRESS of your current machine, for example, on Machine 1 <kbd>listen_address: 192.168.122.100</kbd>
- <kbd>$ nodetool flush -- system</kbd>
- Open the file <kbd>$ sudo nano /etc/cassandra/conf/cassandra.yaml</kbd>
- 

## 5. Usefull Commands to Cassandra
### 5.1 Renaming the Cluster
- <kbd>$ cqls</kbd>
- <kbd>cqlsh> update system.local set cluster_name = 'new_cluster_name' where key='local'</kbd>
- <kbd>cqls> exit</kbd>
- <kbd>$ nodetool flush -- system</kbd>
