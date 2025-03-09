# Jenkins CI/CD Pipeline Setup

This guide outlines the steps to set up a **Jenkins CI/CD pipeline** with **slave nodes**, **Tomcat deployment**, and **Nexus integration** on an AWS-based environment.

## 1. Create Jenkins Server

### Step 1: Install Git, Java 1.8, and Maven
```sh
yum install git java-1.8.0-openjdk maven -y
```

### Step 2: Add Jenkins Repository & Import Key
```sh
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

### Step 3: Install Java 11 and Jenkins
```sh
amazon-linux-extras install java-openjdk11 -y
yum install jenkins -y
update-alternatives --config java
```

### Step 4: Start Jenkins Service
```sh
systemctl start jenkins.service
systemctl status jenkins.service
```

---
## 2. Create and Configure Jenkins Slave Nodes

### Step 1: Install Java on the Slave Server
```sh
amazon-linux-extras install java-openjdk11 -y
```

### Step 2: Configure Jenkins Slave
1. Navigate to **Dashboard → Manage Jenkins → Nodes & Clouds → New Node**
2. Set **Node Name:** `slave1`
3. Select **Permanent Agent**, then click **Save**

### Step 3: Configure Slave Properties
- **Number of Executors:** 3
- **Remote Root Directory:** `/tmp`
- **Labels:** `swiggy`
- **Launch Method:** SSH
- **Host:** `<Private IP>`
- **Credentials:** Add Jenkins credentials with SSH private key

### Step 4: Assign Jobs to Slave
1. **Dashboard → Job → Configure**
2. Enable **Restrict where this project can be run**
3. Set **Label Expression:** `slave1`
4. **Save**

---
## 3. Install Tomcat on Slave

### Step 1: Install Java
```sh
amazon-linux-extras install java-openjdk11 -y
```

### Step 2: Download & Extract Tomcat
```sh
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.83/bin/apache-tomcat-9.0.83.tar.gz
tar -zxvf apache-tomcat-9.0.83.tar.gz
```

### Step 3: Configure User Authentication
Edit `tomcat-users.xml` to add roles and users:
```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="raham123" roles="manager-gui,manager-script"/>
```

### Step 4: Modify Tomcat Manager Security
Remove lines **21 & 22** in:
```sh
vim apache-tomcat-9.0.83/webapps/manager/META-INF/context.xml
```

### Step 5: Start Tomcat
```sh
sh apache-tomcat-9.0.83/bin/startup.sh
```

### Step 6: Access Tomcat
- Open **`http://<Public IP>:8080`**
- Login with **Username:** `tomcat`, **Password:** `raham123`

---
## 4. Add Slave Credentials to Jenkins
1. **Manage Jenkins → Credentials → Add Credentials**
2. Enter:
   - **Username:** `tomcat`
   - **Password:** `raham123`
3. Copy credential ID and use it in the pipeline.

---
## 5. Install Nexus and Integrate with Jenkins

### Step 1: Install Required Packages
```sh
sudo yum update -y
sudo yum install wget -y
```

### Step 2: Install Java 1.8
```sh
sudo yum install java-1.8.0-openjdk.x86_64 -y
```

### Step 3: Download and Extract Nexus
```sh
sudo mkdir /app && cd /app
sudo wget -O nexus.tar.gz https://download.sonatype.com/nexus/3/latest-unix.tar.gz
sudo tar -xvf nexus.tar.gz
sudo mv nexus-3* nexus
```

### Step 4: Create Nexus User
```sh
sudo adduser nexus
```

---
## 6. Install "Deploy to Container" Plugin in Jenkins
1. **Manage Jenkins → Tools → Install "Deploy to Container" Plugin**

---
## 7. Write & Run Jenkins Pipeline
Create a `Jenkinsfile` with deployment steps, integrating Jenkins with Tomcat and Nexus for automated CI/CD.

