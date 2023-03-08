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
On the [example of AWS services](https://dzone.com/articles/confused-by-aws-storage-options-s3-ebs-amp-efs-explained) understand the difference between Block Storage, Object Storage and Network File System.

## Setup and technologies used in Project 7

- As a member of a DevOps team, you will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.

- In this project you will implement a solution that consists of following components:

- Infrastructure: AWS
- Webserver Linux: Red Hat Enterprise Linux 8
- Database Server: Ubuntu 20.04 + MySQL
- Storage Server: Red Hat Enterprise Linux 8 + NFS Server
- Programming Language: PHP
- Code Repository: [GitHub](https://github.com/darey-io/tooling)

- On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using [Network File Sytem (NFS)](https://en.wikipedia.org/wiki/Network_File_System) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.

<img width="646" alt="Tooling website infrastructure" src="https://user-images.githubusercontent.com/115954100/223786364-22d16742-1ce7-4126-8a97-947dbebeaa03.png">

- It is important to know what storage solution is suitable for what use cases, for this – you need to answer following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Base on this you will be able to choose the right storage system for your solution.


## STEP 1 – PREPARE NFS SERVER

1. - Spin up a new EC2 instance with RHEL Linux 8 Operating System.

2. - Based on your LVM experience from **Project 6**, Configure 3 LVM on the Server.

<img width="593" alt="3 Logical volumes created" src="https://user-images.githubusercontent.com/115954100/223803569-69a6c843-8df4-43e3-bce4-b28a35008937.png">

- Instead of formating the disks as ***ext4*** you will have to format them as ***xfs***

<img width="515" alt="formatted logical volume with xfs filesystem" src="https://user-images.githubusercontent.com/115954100/223803716-5e4e18a6-181e-4d9f-9b1b-db3ada0ae713.png">

3. - Ensure there are 3 **Logical Volumes**. ***lv-opt lv-apps, and lv-logs***

Create mount points on **/mnt** directory for the logical volumes as follow:
- Mount **lv-apps** on **/mnt/apps** – To be used by webservers
- Mount **lv-logs** on **/mnt/logs** – To be used by webserver logs
- Mount **lv-opt** on **/mnt/opt** – To be used by Jenkins server in *Project 8*

4. - Install NFS server, configure it to start on reboot and make sure it is u and running

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

<img width="754" alt="NFS Server enabled and active" src="https://user-images.githubusercontent.com/115954100/223804143-51ff9c02-34da-4405-a0a3-1bdfc51bbc38.png">

5. - Export the mounts for webservers’ ***subnet cidr*** to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security.
To check your ***subnet cidr*** – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

<img width="752" alt="cidr" src="https://user-images.githubusercontent.com/115954100/223799749-426c902f-8091-4c1f-b89b-e9ba395dd97c.png">

- Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```

- Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.48.0/20 ):
- 
<img width="378" alt="configured access to NFS for clients" src="https://user-images.githubusercontent.com/115954100/223801859-4fd8d35b-0f7a-480d-b2f1-a606f70d4945.png">





