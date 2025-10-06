# Three-Tier-Architecture-on-AWS-using-VPC
Secure 3-tier AWS architecture with a proxy-server in public subnet, app &amp; db server in private subnet, &amp; RDS used to student registration database.
## About the Project:

The objective of this project is to design and implement a secure and scalable **three-tier architecture** on Amazon Web Services (AWS) using **Virtual Private Cloud (VPC)**. The architecture separates the infrastructure into three logical layers:

1. **Proxy Layer** (Jump Server) in the public subnet
2. **Application Layer** in a private subnet
3. **Database Layer** in another private subnet (using EC2 or Amazon RDS)

This setup reflects real-world enterprise deployments where security, isolation, and controlled access are critical.

## Technologies Used:

- **AWS VPC** →  Isolated cloud network to host resources securely
- **EC2** → Virtual servers to run applications and databases
- **Subnets** → Logical divisions of VPC for public/private access
- **Internet Gateway** → Enables internet access for public subnet
- **Route Tables** → Controls traffic flow within VPC
- **Security Groups** → Firewall rules for EC2 instances
- **SSH** → Secure protocol to access servers remotely
- **IAM (optional)** → Manages user permissions and roles in AWS

## Prerequisites:

- AWS Free Tier account
- Basic knowledge of networking (IP, CIDR)
- Familiarity with EC2 and AWS Console
- SSH key pair for EC2 login
- Optional: AWS CLI or Terraform for automation

---

## What is VPC?

A **Virtual Private Cloud** is a logically isolated section of AWS where you can launch resources in a defined network. You control IP ranges, subnets, route tables, and gateways.

## What is a Subnet?

A **Subnet** is a segment of a VPC’s IP range.

- **Public Subnet**: Connected to the internet via an Internet Gateway
- **Private Subnet**: Isolated, no direct internet access

## What is an Internet Gateway?

An **IGW** allows communication between your VPC and the internet. It’s attached to the VPC and used in route tables for public access.

## What is a Route Table?

A **Route Table** defines how traffic flows within your VPC.

- Public route table: routes 0.0.0.0/0 to IGW
- Private route table: no internet route (or via NAT if needed)

## What is a NAT Gateway?

A **NAT Gateway** allows instances in a private subnet to access the internet **without being exposed** to incoming traffic.
(Not used in this project, but useful for updates or outbound access)

## What is a Security Group?

A **Security Group** acts as a virtual firewall for EC2 instances.

- App SG: allows SSH, HTTP
- DB SG: allows SSH/MySQL only from App SG

---

## Step 1: Create a VPC

1. Go to AWS console → Open VPC Service 
2. Click on Create VPC 
    - Select → VPC only
    - Name → **three-tier-project**
    - **IPv4 CIDR → 10.0.0.0/16**
    - Click on Create VPC

![Project Screenshot](/images/vpc-done.png).

## Step 2: Create a Subnet’s

1. Go to Subnet Option 
2. Click on Create Subnet 
    - Select VPC → **three-tier-project**

![Project Screenshot](/images/subnet-create.png)

- Create Subnet 1
    - Name → Public Subnet
    - Availability Zone → United States (N. Virginia) / use1-az4 (us-east-1a)
    - IPv4 subnet CIDR block → 10.0.0.0/20

![Project Screenshot](/images/subnet-pub.png)

- **Add new subnet**
- Create Subnet 2
    - Name → Private Subnet-1
    - Availability Zone → United States (N. Virginia) / use1-az4 (us-east-1b)
    - IPv4 subnet CIDR block → 10.0.16.0/20

![Project Screenshot](/images/subnet-pri-1.png)

- Create Subnet 3
    - Name → Private Subnet-2
    - Availability Zone → United States (N. Virginia) / use1-az4 (us-east-1c)
    - IPv4 subnet CIDR block → 10.0.47.0/20

![Project Screenshot](/images/subnet-pri-2.png)
![Project Screenshot](/images/subnet-done.png)

3. Public Subnet to set public IP 
    - Select Public Subnet
    - Click Action → Edit subnet settings
    - Check-in → Auto-assign IP settings
    - Click Save Changes

![Project Screenshot](/images/edit-pub-subnet.png)

## Step 3: Create Internet Gateway

1. Go to VPC service → Click on Internet Gateway Tab
2. Click on **Create internet gateway**
    - Name → **3-tier-IGW**
    - Click on Create internet gateway

![Project Screenshot](/images/IGW-create.png)

3. Select internet gateway
4. Click on Action → Attach to VPC 
    - Select VPC → **three-tier-project**
    - click on Attach internet gateway

![Project Screenshot](/images/IGW-Attach.png)

5. IGW - Entry on Route Table
    - Click on Route Table Tab
    - Select your Route Table (main - Public-RT)
    - Click To Routes → Edit Routes
    - Add Route → Select internet gateway → Select IGW-current your
    - Click on Save Changes
6. Your Internet Gateway is Created 

![Project Screenshot](/images/IGW-entry-pub-1.png)
![Project Screenshot](/images/IGW-entry-pub-2.png)

## Step 4: Create NAT Gateway

1. Click on the NAT Gateway Tab 
2. Click Create NAT Gateway
    - NAT gateway settings
    - Name → **3-tier-NAT**
    - Subnet → Public-Subnet
    - Elastic IP → Click on **Allocate Elastic IP**
    - Click on Create NAT Gateway

![Project Screenshot](/images/NAT-create.png)

3. Create a Private Route Table 
    - Click on Route Table Tab
    - Click on Create Route Table
    - Name → Private-RT
    - Click on Save Changes

![Project Screenshot](/images/Route-create-pri.png)

4. NAT - Entry on Route Table
    - Click on Route Table Tab
    - Select your Route Table (Private-RT)
    - Click To Routes → Edit Routes
    - Add Route →Select NAT-current your
    - Click on Save Changes

![Project Screenshot](/images/NAT-entry-pri-1.png)
![Project Screenshot](/images/NAT-entry-pri-2.png)

5. Edit Subnet Associations
    - Click on Route Table Tab
    - Select your Route Table (Private-RT)
    - Click To Routes → Edit Subnet Associations
    - Select Private-Subnet 1 & 2 (both)
    - Click on Save Changes

![Project Screenshot](/images/edit-subnet-associ-1.png)
![Project Screenshot](/images/edit-subnet-associ-2.png)

## Step 5: Create a Security Group

1. Go to AWS Console → Go to EC2 service
2. Click Security Groups under Network & Security
3. Click Create Security Group
    - Name: 3-tier-SG
    - Description: Security Group for 3-tier instances
    - VPC: (default-VPC)
    - Under Inbound Rules, click “Add Rule”:
        - Add: SSH, HTTP, HTTPS, etc
        - Port Range: 22,8080,3306,80, etc
    - Under Outbound Rules, keep default (all traffic allowed) or customize
    - Click Create security group
4. Now your 3-tier-SG security group created

![Project Screenshot](/images/SGroup-1.png)
![Project Screenshot](/images/SGroup-2.png)

## Step 6: Launch EC2 Instances

1. Go to AWS Console → EC2
2. Click on Create Instance 
    - Name → **proxy_server**
    - Choose AMI → Amazon Linux.
    - Instance type → t2.micro
    - Key pair → pem-server-key
    - Network settings Edit
    - Select VPC → **three-tier-project**
    - Subnet → Public-Subnet
    - Auto-assign public IP → **Enable**
    - security group → 3-tier-SG

![Project Screenshot](/images/proxy-server-1.png)
![Project Screenshot](/images/proxy-server-2.png)

3. Click on Create Instance 
    - Name → **app_server**
    - Choose AMI → Amazon Linux.
    - Instance type → t2.micro
    - Key pair → pem-server-key
    - Network settings Edit
    - Select VPC → **three-tier-project**
    - Subnet → Private-Subnet-1
    - Auto-assign public IP → **Disable**
    - security group → 3-tier-SG

![Project Screenshot](/images/app-server-1.png)
![Project Screenshot](/images/app-server-2.png)

4. Click on Create Instance 
    - Name → **db_server**
    - Choose AMI → Amazon Linux.
    - Instance type → t2.micro
    - Key pair → pem-server-key
    - Network settings Edit
    - Select VPC → **three-tier-project**
    - Subnet → Private-Subnet-2
    - Auto-assign public IP → **Disable**
    - security group → 3-tier-SG
    - Click on Create

![Project Screenshot](/images/db-server-1.png)
![Project Screenshot](/images/db-server-2.png)
![Project Screenshot](/images/instance-done.png)

## Step 7: Create a RDS Database

1. Go to AWS Console → Go to Aurora and RDS Services 
2. Click on the Database Tab → Click to Create Database   
    - Choose a database creation method → Standard create
    - Engine options (Engine type)→ **MariaDB** (version - MariaDB 11.4.5)
    - **Templates → Free tier**
    - DB instance identifier → **three-tier-rds**
    - Credentials Settings
        - Master username → admin
        - Check-in → **Auto generate password**
    - Virtual private cloud (VPC) → **three-tier-project**
    - Existing VPC security groups →3-tier-SG
    - Database authentication → Password authentication
    - Create Database

![Project Screenshot](/images/RDS-1.png)
![Project Screenshot](/images/RDS-2.png)
![Project Screenshot](/images/RDS-3.png)

3. Now RDS Database is Created 
4. copy password & endpoint 

```sql
#password 
TzY3FyTEFq2MVSuPlBe7
#endpoint 
three-tier-rds.c8taowgcq72q.us-east-1.rds.amazonaws.com
```

![Project Screenshot](/images/RDS-done.png)
![Project Screenshot](/images/RDS-pass-endpoint.png)

## Step 6: Change instances names for understand

1. change the all instance names set 
2. connect to proxy server

```bash
ssh -i "pem-server-key.pem" ec2-user@13.221.234.198
sudo hostnamectl hostname proxy-server
exit

#copy key local machine to proxy-server
scp -i /c/ssh-keys/pem-server-key.pem /c/ssh-keys/pem-server-key.pem ec2-user@13.221.234.198:/home/ec2-user/
ls
```

![Project Screenshot](/images/connect-proxy.png)

3. Connect proxy server to **app-server** 

```bash
sudo ssh -i "pem-server-key.pem" ec2-user@10.0.23.125
sudo hostnamectl hostname app-server
exit;
```

![Project Screenshot](/images/connect-app.png)

4. Connect proxy server to **db-server** 

```bash
sudo ssh -i "pem-server-key.pem" ec2-user@10.0.43.98
sudo hostnamectl hostname db-server
exit;
```

![Project Screenshot](/images/connect-db.png)

## Step 7: Configure proxy-server

1. Connect to proxy server 

```bash
ssh -i "pem-server-key.pem" ec2-user@13.221.234.198
```

2. update system & install/start nginx

```bash
sudo yum update
sudo yum install nginx -y 
sudo systemctl start nginx 
sudo systemctl enable nginx 
```

![Project Screenshot](/images/proxy-setup-1.png)

3. configure nginx.conf file 

```bash
cd /etc/nginx/
sudo vim nginx.conf
#write follwing code in the server block
location /{
	proxy_pass http://10.0.23.125:8080;
}
:wq
systemctl restart nginx
logout 
```

![Project Screenshot](/images/proxy-setup-2.png)

## Step 8: Application server setup

1. connect to proxy-server to app-server 

```bash
ls
# pem-server-key
#showing key that are already copy on step 7
#Connect proxy server to **app-server**
sudo ssh -i "pem-server-key.pem" ec2-user@10.0.23.125
```

2. update system & install/start java & tomcat 

```bash
sudo -i 
yum update
yum install java -y 
java --version                      # showing java version 
# now install tomcat server 
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.109/bin/apache-tomcat-9.0.109.tar.gz
ls                                  # file showing in tar format 
tar -xvzf apache-tomcat-9.0.109.tar.gz
#now untared
rm -rf apache-tomcat-9.0.109.tar.gz
#removed-untar-file
ls
mv apache-tomcat-9.0.109/ apache/
ls                                  # apache 
mv apache/ /opt/
cd /opt/apache/
sudo chmod +x bin/
#for the permission to file
cd /bin
ls                                  # catalina.sh 
sudo ./catalina.sh start            #tomacat start 
cd .. 
ls                                  # webapps
cd /webapps
ls
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/student.war
ls 
sudo unzip student.war -d student   # unzip file                           
cd student/                         # student
ls                                  # showing all your code 
cd .. 
cd .. 
exit 
exit
```

3. Now go to proxy-server

```bash
cd /etc/nginx/
sudo vim nginx.conf
#write follwing code in the server block
location /{
	proxy_pass http://10.0.23.125:8080/student/;
}
:wq
sudo systemctl restart nginx
#copy proxy-server-ip paste to run : working form
cd ..
```

![Project Screenshot](/images/app-setup-1.png)

## Step 9: Now db-server & RDS setup

1. Connect to proxy-server to db-server 

```bash
ls
#showing key that are already copy on step 7
#Connect proxy server to **db-server**
sudo ssh -i "pem-server-key.pem" ec2-user@10.0.43.98
```

2. update system & install/start mariaDB 

```bash
sudo yum update
sudo yum install mariadb105-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

![Project Screenshot](/images/db-setup-1.png)

3. Connect to RDS database & database

```bash
sudo mysql -h three-tier-rds.c8taowgcq72q.us-east-1.rds.amazonaws.com -u admin -p
show databases; 
create database studentapp; 
```

![Project Screenshot](/images/db-setup-2.png)

4. Create table using below query 

```bash
use studentapp;
CREATE TABLE if not exists students(student_id INT NOT NULL AUTO_INCREMENT,
student_name VARCHAR(100) NOT NULL,
student_addr VARCHAR(100) NOT NULL,
student_age VARCHAR(3) NOT NULL,
student_qual VARCHAR(20) NOT NULL,
student_percent VARCHAR(10) NOT NULL,
student_year_passed VARCHAR(10) NOT NULL,
PRIMARY KEY (student_id)
);
exit;
exit
```

![Project Screenshot](/images/db-setup-3.png)

## Step 10: install connector

1. Connect proxy-server to **app-server via ssh**

```bash
sudo ssh -i "pem-server-key.pem" ec2-user@10.0.23.125
```

2. install connector 

```bash
sudo -i 
cd /opt/apache/lib/
curl -O https://s3-us-west-2.amazonaws.com/studentapi-cit/mysql-connector.jar
ls
cd .. 
ls 
cd conf/
ls
```

![Project Screenshot](/images/setup-connector-1.png)

3. edit context.xml file 

```bash
vim context.xml
#copy and paste same as only change blow fileds 
username -> admin
#<rds-password>
password -> TzY3FyTEFq2MVSuPlBe7
# <rds-endpoint>
endpoint -> three-tier-rds.c8taowgcq72q.us-east-1.rds.amazonaws.com
```

<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource" 
maxTotal="500" maxIdle="30" maxWaitMillis="1000" username="admin" password="12345678" 
driverClassName="com.mysql.jdbc.Driver" 
url="jdbc:mysql://endpoint:3306/studentapp?useUnicode=yes&amp;characterEncoding=utf8"/>

![Project Screenshot](/images/setup-connector-2.png)

```bash
cd /opt/apache/bin/
./catalina.sh stop 
./catalina.sh start
#tomcat started  
```

## Step 11: Checking application is running

1. Copy proxy-server ip then enter  to run open your from that fil details then register

```bash
http://13.221.234.198
#submit from
```

![Project Screenshot](/images/run-from-1.png)
![Project Screenshot](/images/run-from-2.png)

2. Check in the Database to details all are submitted or not 

```sql
#open db-server & login into rds database
use studentapp;
show tables;
# students
select *from students; 
```

![Project Screenshot](/images/data-store.png)
![Project Screenshot](/images/data-store2.png)

## Step 12: Delete All instance, NAT, IGW, VPC, RDS

3. Go to EC2 service 
4. Select instance you want to deleted 
5. click on the **Instance state →** Click on Terminate instance 

![Project Screenshot](/images/instance-delete.png)

6. Go to VPC service 
7. Click on NAT Gateway → Select NAT 
8. Click on Delete NAT Gateway 

![Project Screenshot](/images/NAT-delete.png)

9. Click on **Elastic IPs →** Select IP 
10. Click on Release Elastic IP addresses → Click Release 

![Project Screenshot](/images/Elastic-ip-delete.png)

11. Click on Internet Gateway → Select which are delete 
12. Then Click on Action → Detach 1st

![Project Screenshot](/images/detach-IGW.png)
![Project Screenshot](/images/delete-IGW.png)

13. Go to RDS Console 
14. Select RDS which you want to delete 
15. Click on Action → Delete 
    - uncheck the box - snapshotDetermines & backupsDetermines
    - then Check-in the acknowledge
    - type delete me then Click Delete

![Project Screenshot](/images/delete-RDS.png)

16. Re-select then click Action → Delete Internet Gateway 
17. Go to VPC → Select VPC 
18. Click on Action → select **Delete VPC**
19. You deleted VPC then automatically 5 resources will also be deleted
    - security group of vpc, public & private Subnet, then also public & private Route table
20. type delete then Click on Delete

![Project Screenshot](/images/delete-VPC.png)
