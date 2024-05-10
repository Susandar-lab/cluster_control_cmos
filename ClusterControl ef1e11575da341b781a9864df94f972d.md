# ClusterControl

Created by: Susandar Htet
Created time: September 8, 2023 10:46 AM
Tags: Databases

ClusterControl is an agentless management and automation software for database clusters. It helps deploy, monitor, manage and scale your database server/cluster directly from the ClusterControl user interface.



## Requirements

Hardware

ollowing is the minimum system requirement for the ClusterControl host:

- Architecture: x86_64 only
- RAM: >2 GB
- CPU: >2 cores
- Disk space: >40 GB
- Tested cloud platform:
    - AWS EC2
    - Google Cloud
    - Microsoft Azure
    - Digital Ocean
- Internet connection (for selected cluster deployment)

The above should be able to manage and monitor a three-node database cluster smoothly.

Operating System

ClusterControl has been tested on the following operating systems:

- Red Hat Enterprise Linux 7.x/8.x/9.x
- Rocky Linux 8.x/9.x
- AlmaLinux 8.x/9.x
- Ubuntu 18.04/20.04 LTS
- Debian 10.x/11.x

Supported Databases

reference this link >> [https://docs.severalnines.com/docs/clustercontrol/requirements/supported-databases/](https://docs.severalnines.com/docs/clustercontrol/requirements/supported-databases/)

ClusterControl has been tested on the following operating systems:

- Red Hat Enterprise Linux 7.x/8.x/9.x
- Rocky Linux 8.x/9.x
- AlmaLinux 8.x/9.x
- Ubuntu 18.04/20.04 LTS
- Debian 10.x/11.x

Firewall and Security Groups

ClusterControl requires ports used by the following services to be opened/enabled:

- ICMP (echo reply/request).
- SSH (default is 22).
- HTTP (default is 80).
- HTTPS (default is 443).
- MySQL (default is 3306).
- CMON RPC (default is 9500).
- CMON RPC TLS (default is 9501).
- CMON Events (default is 9510).
- CMON SSH (default is 9511).
- CMON Cloud (default is 9518).
- Streaming port for backups through socat/netcat (default is 9999).

## Host Configurations

Set up host definition file in /etc/hosts

```bash
127.0.0.1	localhost
127.0.1.1	ubuntu
172.16.251.99 maria-db01
172.16.250.216 maria-db02

172.16.251.155 qa-nc-db-01
172.16.251.156 qa-nc-db-02
```

Note : hostname of Galera must be the same name from **galera.conf**

## **Operating System User**

ClusterControl controller (cmon) process requires a dedicated operating system user to perform various management and monitoring commands on the managed nodes. The value of `os_user` or `sshuser` in CMON configuration file, must exist on all managed nodes and it should have the ability to perform super-user commands.

You are recommended to install ClusterControl as ‘root’, and running as root is the easiest option. If you perform the installation using another user other than ‘root’, the following must be true:

- The OS user must exist on all nodes
- The OS user must not be `mysql`
- The `sudo` program must be installed on all hosts
- The OS user must be allowed to do sudo, i.e, it must be in sudoers
- The OS user must be configured with the proper PATH environment variable. The following environment `PATH` is expected for the user `myuser`: `PATH=/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/myuser/.local/bin:/home/myuser/bin`

Note that : : ClusterControl requires full access of sudo (all commands) for full functionality. Restricting the commands would cause some of the operations to fail (cluster recovery, failover, backup restoration, service control and cluster deployment).

## Passwordless SSH

Proper passwordless SSH setup from ClusterControl node to all nodes (including ClusterControl node) is mandatory. Before performing any operation on the managed node, the node must be accessible via SSH without using a password by using the key-based authentication instead.

Note : Take note that ClusterControl is fully tested with the RSA public key. Other supported key types should work in most cases.

Setting up Passwordless SSH

To generate an SSH key, use the `ssh-keygen` command which is available with the OpenSSH-client package. On ClusterControl node:

```bash
$ whoami
root
$ ssh-keygen -t rsa # press Enter on all prompts
```

The above command will generate SSH RSA private and public keys under the user’s home directory, `/root/.ssh/`. The private key, `id_rsa` has to be kept secure on the node. The public key, `id_rsa.pub` should be copied over to all nodes that want to be accessed by ClusterControl passwordlessly.

The next step is to copy the SSH public key to all nodes. You may use the `ssh-copy-id` command to achieve this if the destination node supports password authentication:

```bash
$ whoami
root
$ ls -1 ~/.ssh/id*
/root/.ssh/id_rsa
/root/.ssh/id_rsa.pub
$ ssh-copy-id 192.168.0.10 # specify the root password of 192.168.0.10 if prompted
```

The command `ssh-copy-id` will simply copy the public key from the source server and add it into the destination server’s authorized key list, default to `~/.ssh/autohorized_keys` of the authenticated SSH user. If password authentication is disabled, then a manual copy is required. On ClusterControl node, copy the content of SSH public key located at `~/.ssh/id_rsa.pub` and paste it into `~/.ssh/authorized_keys` on all managed nodes (including ClusterControl server).

```bash
$ whoami
root
$ ssh-keygen -t rsa # press Enter on all prompts
$ ls -1 ~/.ssh/id*
/root/.ssh/id_rsa
/root/.ssh/id_rsa.pub
$ ssh-copy-id 192.168.0.10 # specify the root password of 192.168.0.10 if prompted
$ ssh-copy-id 192.168.0.11 # specify the root password of 192.168.0.11 if prompted
$ ssh-copy-id 192.168.0.12 # specify the root password of 192.168.0.12 if prompted
$ ssh-copy-id 192.168.0.13 # specify the root password of 192.168.0.13 if prompted
```

##All of the cluster nodes have to be ssh login from cluster control nodes

##Copy the cluster control rsa public key to the client nodes under root authorized keys.

## Installation

### Manual Installation

- clustercontrol – ClusterControl web user interface.
- clustercontrol-controller – ClusterControl CMON controller
- clustercontrol-notifications – ClusterControl notification module, if you would like to integrate with third-party tools like PagerDuty and Slack.
- clustercontrol-ssh – ClusterControl web-based SSH module, if you would like to access the host via SSH directly from ClusterControl UI.
- clustercontrol-cloud – ClusterControl cloud module, if you would like to manage your cloud instances directly from ClusterControl UI.
- clustercontrol-clud – ClusterControl cloud file manager module, if you would like to upload and download backups from cloud storage. It requires clustercontrol-cloud.
- s9s-tools – ClusterControl CLI client, if you would like to manage your cluster using a command-line interface.

### Step - 1

Setup APT Repository

Add Severalnines repository public key into your APT keyring:

```bash
$ apt-get -y install gnupg2
$ wget http://repo.severalnines.com/severalnines-repos.asc -O- | sudo apt-key add -
```

You can download the repository definition from [Severalnines download page](http://www.severalnines.com/downloads/cmon/):

```bash
$ sudo wget http://www.severalnines.com/downloads/cmon/s9s-repo.list -P /etc/apt/sources.list.d/
```

Or, add the Severalnines APT source list manually:

```bash
$ echo 'deb [arch=amd64] http://repo.severalnines.com/deb ubuntu main' | sudo tee /etc/apt/sources.list.d/s9s-repo.list
```

Update package list:

```bash
$ sudo apt-get update
```

Look for ClusterControl packages:

```bash
$ sudo apt-cache search clustercontrol
```

### Step - 2

Setup ClusterControl CLI repository – [Debian/Ubuntu DEB Repositories](https://docs.severalnines.com/docs/clustercontrol/user-guide-cli/installation/#package-manager-yum-apt).

To install, simply do :

```bash
$ wget -qO - http://repo.severalnines.com/s9s-tools/$(lsb_release -sc)/Release.key | sudo apt-key add -
$ echo "deb http://repo.severalnines.com/s9s-tools/$(lsb_release -sc)/ ./" | sudo tee /etc/apt/sources.list.d/s9s-tools.list
$ sudo apt-get update
$ sudo apt-get install s9s-tools
$ s9s --help
```

Compile From Source

================

To build from source, you may require additional packages and tools to be installed:

1. Get the source code from Github:

```bash
$ git clone https://github.com/severalnines/s9s-tools.git
```

2. Navigate to the source code directory:

```bash
$ cd s9s-tools
```

3. You may need to install development packages such as C/C++ compiler, autotools, openssl-devel etc:

```bash
# RHEL/CentOS
$ yum groupinstall "Development Tools"
$ yum install automake git openssl-devel

# Ubuntu/Debian
$ sudo apt-get install build-essential automake git libssl-dev byacc flex bison
```

4. Compile the source code:

```bash
$ ./autogen.sh
$ ./configure
$ make
$ make install
$ s9s --help
```

### Step - 3

If you have AppArmor running, disable it and open the required ports (or stop iptables):

```bash
$ sudo /etc/init.d/apparmor stop
$ sudo /etc/init.d/apparmor teardown
$ sudo update-rc.d -f apparmor remove
$ sudo service iptables stop
```

### Step - 4

Install ClusterControl dependencies:

```bash
$ sudo apt-get update
$ sudo apt-get install -y curl apache2 libapache2-mod-php mailutils dnsutils mysql-client mysql-server php-common php-mysql php-gd php-ldap php-curl php-json
```

### Step - 5

Install the ClusterControl controller package:

```bash
$ sudo apt-get install -y clustercontrol-controller \
  clustercontrol \
  clustercontrol-ssh \
  clustercontrol-notifications \
  clustercontrol-cloud \
  clustercontrol-clud \
  s9s-tools
```

### Step - 6

Create two databases called cmon and dcps and grant user cmon:

```bash
$ mysql -uroot -p -e 'DROP SCHEMA IF EXISTS cmon; CREATE SCHEMA cmon'
$ mysql -uroot -p -e 'DROP SCHEMA IF EXISTS dcps; CREATE SCHEMA dcps'
$ mysql -uroot -p -e 'GRANT ALL PRIVILEGES ON *.* TO "cmon"@"localhost" IDENTIFIED BY "{cmonpassword}" WITH GRANT OPTION'
$ mysql -uroot -p -e 'GRANT ALL PRIVILEGES ON *.* TO "cmon"@"127.0.0.1" IDENTIFIED BY "{cmonpassword}" WITH GRANT OPTION'
$ mysql -uroot -p -e 'FLUSH PRIVILEGES'
```

### Step - 7

For Apache 2.4 and later (Ubuntu 14.04/Debian 8 and later), the default document root is `/var/www/html`. Create a symbolic link for the components:

```bash
$ ln -sfn /var/www/clustercontrol /var/www/html/clustercontrol
$ ln -sfn /var/www/cmon /var/www/html/cmon
```

### Step - 8

$ mysql -uroot -p cmon < /usr/share/cmon/cmon_db.sql
$ mysql -uroot -p cmon < /usr/share/cmon/cmon_data.sql
$ mysql -uroot -p dcps < /var/www/clustercontrol/sql/dc-schema.sql

### Step - 9

9. Generate a ClusterControl key to be used by `RPC_TOKEN` and `rpc_key`:

```bash
$ uuidgen | tr -d '-'
6856d96a19d049aa8a7f4a5ba57a34740b3faf57
```

Note >> Rememer that key

And create the ClusterControl Controller (cmon) configuration file at `/etc/cmon.cnf` with the following configuration options:

```bash
mysql_port=3306
mysql_hostname=127.0.0.1
mysql_password={cmonpassword}
hostname={ClusterControl primary IP address}
rpc_key={ClusterControl API key as generated above}
```

Example : 

```bash
$ cat /etc/cmon.cnf
#
# clustercontrol-controller configuration file
# Copyright 2016 severalnines.com
#

## CMON database config - mysql_password is for the 'cmon' user
mysql_port=3306
mysql_hostname=127.0.0.1
mysql_password=adminsecure
hostname=172.16.251.45
rpc_key=22257f63cdca4c7797c55b9777feb5b4

## hostname is the hostname of the current host
hostname=

## The default logfile
logfile=/var/log/cmon.log

## For possible access restriction
# rpc_key = DEADBEEF01234567ABCDEF
#
controller_id=7c3b658f-ccf9-4fb6-af17-99f816ab5c08
```

Note : The value of `hostname` must be either a valid FQDN or IP address of the ClusterControl node. If the host has multiple IP addresses, pick the primary IP address of the host.

### Step - 10

ClusterControl’s event and cloud modules require `/etc/default/cmon` for service definition. Create the file and add the following lines:

```bash
EVENTS_CLIENT="http://127.0.0.1:9510"
CLOUD_SERVICE="http://127.0.0.1:9518"
```

### Step - 11

Copy the provided Apache configuration files to their locations, create symlinks to the configuration files and prepare SSL key and certificate:

```bash
$ cp -f /var/www/clustercontrol/ssl/server.crt /etc/ssl/certs/s9server.crt
$ cp -f /var/www/clustercontrol/ssl/server.key /etc/ssl/certs/s9server.key
$ rm -rf /var/www/clustercontrol/ssl
$ cp -f /var/www/clustercontrol/app/tools/apache2/s9s.conf /etc/apache2/sites-available/
$ cp -f /var/www/clustercontrol/app/tools/apache2/s9s-ssl.conf /etc/apache2/sites-available/
$ rm -f /etc/apache2/sites-enabled/000-default.conf
$ rm -f /etc/apache2/sites-enabled/default-ssl.conf
$ rm -f /etc/apache2/sites-enabled/001-default-ssl.conf
$ ln -sfn /etc/apache2/sites-available/s9s.conf /etc/apache2/sites-enabled/001-s9s.conf
$ ln -sfn /etc/apache2/sites-available/s9s-ssl.conf /etc/apache2/sites-enabled/001-s9s-ssl.conf
$ sed -i 's|^[ \t]*SSLCertificateFile.*|SSLCertificateFile /etc/ssl/certs/s9server.crt|g' /etc/apache2/sites-available/s9s-ssl.conf
$ sed -i 's|^[ \t]*SSLCertificateKeyFile.*|SSLCertificateKeyFile /etc/ssl/certs/s9server.key|g' /etc/apache2/sites-available/s9s-ssl.conf
```

### Step - 12

Enable required Apache modules and create a symlink to sites-enabled for default HTTPS virtual host:

```bash
$ a2enmod ssl rewrite proxy proxy_http proxy_wstunnel
$ a2ensite default-ssl
```

### Step - 13

Rename the ClusterControl UI default file and assign correct permission to the file:

```bash
$ mv /var/www/clustercontrol/bootstrap.php.default /var/www/clustercontrol/bootstrap.php
$ chmod 644 /var/www/clustercontrol/bootstrap.php
```

### Step - 14

Assign correct ownership and permissions:

```bash
$ chmod -R 777 /var/www/html/clustercontrol/app/tmp
$ chmod -R 777 /var/www/html/clustercontrol/app/upload
$ chown -Rf www-data.www-data /var/www/html/clustercontrol/
```

### Step - 15

Use the generated value from step #9 and specify it in `/var/www/clustercontrol/bootstrap.php` under the `RPC_TOKEN` constant and configure MySQL credentials for the ClusterControl UI by updating the `DB_PASS` and `DB_PORT` constants with the cmon user password and MySQL port for dcps database:

```bash
define('DB_PASS', '{cmonpassword}');
define('DB_PORT', '3306');
define('RPC_TOKEN', '{Generated ClusterControl API token}');
```

Note : Replace `{cmonpassword}` and `{Generated ClusterControl API token}` with appropriate values.

### Step - 16

Insert the generated API token from step #9 into `dcps.apis` table, so ClusterControl UI can use the security token to retrieve cluster information from the controller service:

```bash
$ mysql -uroot -p -e "INSERT IGNORE INTO dcps.apis (id, company_id, user_id, url, token, created) values (1,1,1,'http://127.0.0.1','{generated ClusterControl API token}', UNIX_TIMESTAMP())"
```

### Step - 17

Restart Apache webserver to apply the changes:

```bash
$ sudo service apache2 restart
```

### Step - 18

Enable ClusterControl on boot and start them:

For sysvinit/upstart:

```bash
$ sudo update-rc.d cmon defaults
$ sudo update-rc.d cmon-ssh defaults
$ sudo update-rc.d cmon-events defaults
$ sudo update-rc.d cmon-cloud defaults
$ service cmon start
$ service cmon-ssh start
$ service cmon-events start
$ service cmon-cloud start
```

For systemd:

```bash
$ systemctl enable cmon cmon-ssh cmon-events cmon-cloud
$ systemctl restart cmon cmon-ssh cmon-events cmon-cloud
```

### Step -19

Create the ccrpc user which is required since the ClusterControl version 1.8.2 to support new user management

```bash
$ export S9S_USER_CONFIG=$HOME/.s9s/ccrpc.conf
$ s9s user --create --new-password={generated ClusterControl API token} --generate-key --private-key-file==$HOME/.s9s/ccrpc.key --group=admins --controller=https://localhost:9501 ccrpc
$ s9s user --set --first-name=RPC --last-name=API --cmon-user=ccrpc &>/dev/null
```

### Step - 20

Generate an SSH key to be used by ClusterControl when connecting to all managed hosts. In this example, we are using the ‘root’ user to connect to the managed hosts. To generate an SSH key for the root user, do:

```bash
$ whoami
root
$ ssh-keygen -t rsa # Press enter for all prompts
```

Note : If you are running as sudoer, the default SSH key will be located under `/home/$USER/.ssh/id_rsa`. See [Operating System User](https://docs.severalnines.com/docs/clustercontrol/requirements/operating-system-user/).

### Step - 21

Before importing a database server/cluster into ClusterControl or deploy a new cluster, set up passwordless SSH from ClusterControl host to the database host(s). Use the following command to copy the SSH key to the target hosts:

```bash
$ ssh-copy-id -i ~/.ssh/id_rsa {SSH user}@{IP address of the target node}
```

Note : Replace {SSH user} and {IP address of the target node} with appropriate values. Repeat the command for all target hosts.

### Step - 22

Open ClusterControl UI at `https://{ClusterControl_host}/clustercontrol` and create the default admin password by providing a valid email address and password. You will be redirected to the ClusterControl default page.

The installation is complete and you can start to import existing or deploy a new database cluster. Please review the [User Guide (GUI)](https://docs.severalnines.com/docs/clustercontrol/user-guide-gui/) for details.

REFERENCE >> [https://docs.severalnines.com/docs/clustercontrol/](https://docs.severalnines.com/docs/clustercontrol/)

ISSUES

======

If error happens like cannot connect mysql database, check DB_PASS from `/var/www/clustercontrol/bootstrap.php`

Log File >> /var/log/cmon.log

mysql
cat /etc/cmon.cnf | grep ^mysql_password
cat /etc/cmon.d/cmon_1.cnf | grep ^mysql_password
cat /etc/cmon.d/cmon_2.cnf | grep ^mysql_password
cd /etc/cmon.d/
ls
cat /etc/cmon.d/cmon_6.cnf | grep ^mysql_password
cat /etc/cmon.d/cmon_8.cnf | grep ^mysql_password
cat /var/www/html/clustercontrol/bootstrap.php | grep DB_PASS

systemctl status cmon
