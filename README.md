##1. CREATE JENKINS SERVER 
#STEP-1: INSTALLING GIT JAVA-1.8.0 MAVEN 
yum install git java-1.8.0-openjdk maven -y


#STEP-2: GETTING THE REPO (jenkins.io --> download -- > redhat)
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key


#STEP-3: DOWNLOAD JAVA11 AND JENKINS
amazon-linux-extras install java-openjdk11 -y
yum install jenkins -y
update-alternatives --config java


#STEP-4: RESTARTING JENKINS (when we download service it will on stopped state)
systemctl start jenkins.service
systemctl status jenkins.service


##2. CREATE 2 SLAVES AND CONFIGURE THEM TO JENKINS
SETUP:
#STEP-1 : Create a server and install java-11
amazon-linux-extras install java-openjdk11 -y


#STEP-2: SETUP THE SLAVE SERVER
Dashboard -- > Manage Jenkins -- > Nodes & Clouds -- > New node -- > nodename: abc -- > permanaent agent -- > save 


CONFIGURATION OF SALVE:


Number of executors : 3 #Number of Parallel builds
Remote root directory : /tmp #The place where our output is stored on slave sever.
Labels : swiggy #place the op in a particular slave
useage: last option
Launch method : last option 
Host: (your privte ip)
Credentials -- > add -- >jenkins -- > Kind : ssh username with privatekey -- > username: ec2-user 
privatekey : pemfile of server -- > save -- > 
Host Key Verification Strategy: last option


DASHBOARD -- > JOB -- > CONFIGURE -- > RESTRTICT WHERE THIS JOB RUN -- > LABEL: SLAVE1 -- > SAVE


##3. INSTALL TOMCAT ON SLAVE
SETUP: CREATE A NEW SERVER
INSTALL JAVA: amazon-linux-extras install java-openjdk11 -y


STEP-1: DOWNLOAD TOMCAT (dlcdn.apache.org)
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.83/bin/apache-tomcat-9.0.83.tar.gz


STEP-2: EXTRACT THE FILES
tar -zxvf apache-tomcat-9.0.83.tar.gz


STEP-3: CONFIGURE USER, PASSWORD & ROLES
vim apache-tomcat-9.0.83/conf/tomcat-users.xml


 56   <role rolename="manager-gui"/>
 57   <role rolename="manager-script"/>
 58   <user username="tomcat" password="raham123" roles="manager-gui, manager-script"/>


STEP-4: DELETE LINE 21 AND 22
vim apache-tomcat-9.0.83/webapps/manager/META-INF/context.xml


STEP-5: STARTING TOMCAT
sh apache-tomcat-9.0.83/bin/startup.sh


CONNECTION:
COPY PUBLIC IP:8080 
manager apps -- > username: tomcat & password: raham123

##4. ADD CREDENTIALS OF SLAVE TO JENKINS
manage jenkins -- > credentials -- > add creds -- > username: tomcat & passowrd: raham123 -- > save
copy id and give to pipeline

##5. SETUP NEXUS & INTEGRATE TO JENKINS
Step 1: Login to your Linux server and update the yum packages. Also install required utilities.

sudo yum update -y
sudo yum install wget -y
Step 2: Install OpenJDK 1.8

sudo yum install java-1.8.0-openjdk.x86_64 -y
Step 3: Create a directory named app and cd into the directory.

sudo mkdir /app && cd /app
Step 4: Download the latest nexus. You can get the latest download links fo for nexus from here.

sudo wget -O nexus.tar.gz https://download.sonatype.com/nexus/3/latest-unix.tar.gz
Untar the downloaded file.

sudo tar -xvf nexus.tar.gz
Rename the untared file to nexus.

sudo mv nexus-3* nexus
Step 5: As a good security practice, it is not advised to run nexus service with root privileges. So create a new user named nexus to run the nexus service.

sudo adduser nexus

##6. INSTALL DEPLOY TO CONTAINER PLUGIN
manage jenkins -- > tools -- > install deply to container plugin

##7. WRITE AND RUN PIPELINE
