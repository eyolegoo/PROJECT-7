# PROJECT-7
DEVOPS TOOLING WEBSITE SOLUTION


## DEVOPS TOOLING WEBSITE SOLUTION


- In previous [Project 6](https://www.darey.io/docs/project-6-step-1/) I implemented a WordPress based solution that was filled with content and can be used as a full fledged website or blog. Moving further I added some more value to my solutions that my DevOps team could utilize. I want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.

- The tools I want our team to be able to use are well known and widely used by multiple DevOps teams, so I will introduce a single DevOps Tooling Solution that will consist of:

- 1 [Jenkins](https://www.jenkins.io) - free and open source automation server used to build [CI/CD](https://en.wikipedia.org/wiki/CI/CD) pipelines.

- 2 [Kubernetes](https://kubernetes.io) – an open-source container-orchestration system for automating computer.

- 3 [Jfrog Artifactory](https://jfrog.com/artifactory/) – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.

- 4 [Rancher](https://www.rancher.com/products/rancher) – an open source software platform that enables organizations to run and manage [Docker](https://en.wikipedia.org/wiki/Docker_(software)) and Kubernetes in production.

- 5 [Grafana](https://grafana.com) – a multi-platform open source analytics and interactive visualization web application.

- 6 [Prometheus](https://prometheus.io) – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.

- 7 [Kibana](https://www.elastic.co/kibana/) – Kibana is a free and open user interface that lets you visualize your Elasticsearch [Elasticsearch](https://www.elastic.co/elasticsearch/) data and navigate the [Elastic](https://www.elastic.co/elastic-stack/) Stack.


**Side Self Study**

- Read about [Network-attached storage (NAS)](https://en.wikipedia.org/wiki/Network-attached_storage), [Storage Area Network (SAN)](https://en.wikipedia.org/wiki/Storage_area_network) and related protocols like NFS, (s)FTP, SMB, iSCSI. Explore what [Block-level storage](https://en.wikipedia.org/wiki/Block-level_storage) is and how it is used by Cloud Service providers, know the difference from [Object storage](https://en.wikipedia.org/wiki/Object_storage).

- On the [example of AWS services](https://dzone.com/articles/confused-by-aws-storage-options-s3-ebs-amp-efs-explained) understand the difference between Block Storage, Object Storage and Network File System.

## Setup and technologies used in Project 7

- As a member of a DevOps team, I implemented a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

- In this project I implemented a solution that consists of following components:

- Infrastructure: AWS
- Webserver Linux: Red Hat Enterprise Linux 8
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 8 + NFS Server
- Programming Language: PHP
- Code Repository: [GitHub](https://github.com/darey-io/tooling)

- On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using [Network File Sytem (NFS)](https://en.wikipedia.org/wiki/Network_File_System) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.

<img width="646" alt="Tooling website infrastructure" src="https://user-images.githubusercontent.com/115954100/223786364-22d16742-1ce7-4126-8a97-947dbebeaa03.png">

- It is important to know what storage solution is suitable for what use cases, for this – I answered the following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this I was able to choose the right storage system for my solution.


### STEP 1 – ***PREPARE NFS SERVER***

1. - I Spinned up a new EC2 instance with RHEL Linux 8 Operating System.

2. - Based on my LVM experience from **Project 6**, I Configured 3 LVM on the Server.

<img width="426" alt="New Volume Partition" src="https://user-images.githubusercontent.com/115954100/223806552-70f68ff9-6559-4c46-95c0-2e48392653fb.png">

<img width="667" alt="Lvmdiskscan and Physical volume creation" src="https://user-images.githubusercontent.com/115954100/223807799-c294c7d3-ecc9-4b71-8cd9-946a5f295770.png">

<img width="559" alt="Volume group created" src="https://user-images.githubusercontent.com/115954100/223806619-ccce9454-05b1-4044-a233-53017d4f0c51.png">

<img width="593" alt="3 Logical volumes created" src="https://user-images.githubusercontent.com/115954100/223803569-69a6c843-8df4-43e3-bce4-b28a35008937.png">

- Instead of formating the disks as ***ext4*** I formatted them as ***xfs***

<img width="515" alt="formatted logical volume with xfs filesystem" src="https://user-images.githubusercontent.com/115954100/223803716-5e4e18a6-181e-4d9f-9b1b-db3ada0ae713.png">

3. - I ensured there are 3 **Logical Volumes**. ***lv-opt lv-apps, and lv-logs***

I created mount points on **/mnt** directory for the logical volumes as follow:
- Mount **lv-apps** on **/mnt/apps** – To be used by webservers
- Mount **lv-logs** on **/mnt/logs** – To be used by webserver logs
- Mount **lv-opt** on **/mnt/opt** – To be used by Jenkins server in *Project 8*

4. - I installed NFS server, configured it to start on reboot and make sure it is up and running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

<img width="754" alt="NFS Server enabled and active" src="https://user-images.githubusercontent.com/115954100/223804143-51ff9c02-34da-4405-a0a3-1bdfc51bbc38.png">

5. -I exported the mounts for webservers’ ***subnet cidr*** to connect as clients. For simplicity, I installed all the three Web Servers inside the same subnet, but in production set up I would probably want to separate each tier inside its own subnet for higher level of security.

To check the ***subnet cidr*** – I opened my EC2 details in AWS web console and located ‘Networking’ tab and opened a Subnet link:

<img width="752" alt="cidr" src="https://user-images.githubusercontent.com/115954100/223799749-426c902f-8091-4c1f-b89b-e9ba395dd97c.png">

- I make sure I set up permission that will allow the Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

- I configured access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.48.0/20 ):
- 
<img width="378" alt="configured access to NFS for clients" src="https://user-images.githubusercontent.com/115954100/223801859-4fd8d35b-0f7a-480d-b2f1-a606f70d4945.png">

6. - I checked which port is used by NFS and opened it using Security Groups (add new Inbound Rule)

- `rpcinfo -p | grep nfs` The port is **2049**

- **Important note**: In order for NFS server to be accessible from my client, I also opened the following ports: **TCP 111, UDP 111, UDP 2049**

<img width="810" alt="Port configuration" src="https://user-images.githubusercontent.com/115954100/223809101-b53df945-c467-407c-8a95-ec849dfb9e5a.png">


### STEP 2 — ***CONFIGURE THE DATABASE SERVER***

1. - I installed MySQL server

2. - I created a database and named it **tooling**

3. - I created a database user and named it **webaccess**

4. - I granted permission to **webaccess** user on **tooling** database to do anything only from the webservers **subnet cidr**

<img width="610" alt="DATABASE USERNAME AND PRIVILEGE ON DB SERVER" src="https://user-images.githubusercontent.com/115954100/223824596-596812f1-4878-4afe-827c-dbbdfc84feee.png">


### Step 3 — ***PREPARE THE WEB SERVER***

- I ensured that the Web Servers can serve the same content from shared storage solutions, in this case – NFS Server and MySQL database.
Knowing that one DB can be accessed for **reads** and **writes** by multiple clients. For storing shared files that the Web Servers will use – I utilized NFS and mount previously created Logical Volume **lv-apps** to the folder where Apache stores files to be served to the users **(/var/www)**.

- This approach will make the Web Servers **stateless**, which means I will be able to add new ones or remove them whenever I need, and the integrity of the data (in the database and on NFS) will be preserved.

- During the next steps I did the following:

 - Configured NFS client (this step must be done on all three servers)

 - Deployed a Tooling application to the Web Servers into a shared NFS folder

 - Configured the Web Servers to work with a single MySQL database

1. - I launched a new EC2 instance with RHEL 8 Operating System

2. - I installed NFS client using `sudo yum install nfs-utils nfs4-acl-tools -y`

3. - I mounted **/var/www/** and target the NFS server’s export for apps

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```

4. - I verified that NFS was mounted successfully by running `df -h`. I ensured that the changes will persist on Web Server after reboot:

- `sudo vi /etc/fstab`

- I added the following line inside the file

- `<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

5. - I installed [Remi’s repository](http://www.servermom.org/how-to-enable-remi-repo-on-centos-7-6-and-5/2790/), Apache and PHP

```
sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1
```

<img width="526" alt="Apache running on web server" src="https://user-images.githubusercontent.com/115954100/223834001-1d5545d8-b651-4971-abae-c62db1a70677.png">

- **I repeated steps 1-5 for another 2 Web Servers**.

6. - I verified that Apache files and directories are available on the Web Server in **/var/www** and also on the NFS server in **/mnt/apps**. Seeing the same files – it means NFS is mounted correctly. I created a new file **touch test.txt** from one server and checked if the same file is accessible from other Web Servers.

7. - I located the log folder for Apache on the Web Server and mounted it to NFS server’s export for logs. I repeated step **No 4** to make sure the mount point will persist after reboot.

8. - I forked the tooling source code from [Darey.io Github Account](https://github.com/darey-io/tooling) to my Github account. (Learn how to fork a repo [here](https://www.youtube.com/watch?v=f5grYMXbAV0))

9. - I deployed the tooling website’s code to the Webserver. i ensured that the html folder from the repository is deployed to **/var/www/html**

- **Note 1**: I opened the TCP port 80 on the Web Server.

- **Note 2**: If you encounter 403 Error – check permissions to your **/var/www/html** folder and also disable SELinux `sudo setenforce 0`

- To make this change permanent – I opened following config file `sudo vi /etc/sysconfig/selinux` and set ***SELINUX=disabled** then restarted httpd.

<img width="515" alt="Selinux disabled" src="https://user-images.githubusercontent.com/115954100/223834315-fd765f9d-d7c2-4dac-b53b-aef48c4360b7.png">


10. - I updated the website’s configuration to connect to the database (in **/var/www/html/functions.php** file). Apply **tooling-db.sql** script to the database using this command `mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

11. - I created in MySQL a new admin user with username: **myuser** and password: **password**:

- INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);

12. - I opened the website in my browser **http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php** and ensured I can login into the website with **myuser** user.
  
<img width="883" alt="Congrats" src="https://user-images.githubusercontent.com/115954100/223837093-112266f1-46d2-4155-9e2f-6efb80a7fe66.png">

- TESTING JENKINS 3/19/2023
 
- CHECKING PUBLISH OVER SSH 3/19/2023 
 
- WORKING 
 
- perfect Job


