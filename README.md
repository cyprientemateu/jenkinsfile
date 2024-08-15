This is my hands on repository for practice. 
It's always good to stay up to date with the skills and knowledge. 

![](https://1drv.ms/i/s!Ag3UhH0IOlgnhOw6r1Wd2WORprZzCQ?e=L9LbxH)
###############################################################
##  jenkins master/agent config
###############################################################
**Let us learn more about Jenkins master and slave:**

**Jenkins Master**

Your main Jenkins server is the Master. The Master’s job is to handle:

- Scheduling build jobs.
- Dispatching builds to the slaves for the actual execution.
- Monitor the slaves (possibly taking them online and offline as required).
- Recording and presenting the build results.
- A Master instance of Jenkins can also execute build jobs directly.
**Jenkins Slave**

A Slave is a Java executable that runs on a remote machine. Following are the characteristics of Jenkins Slaves:

- It hears requests from the Jenkins Master instance.
- Slaves can run on a variety of operating systems.
- The job of a Slave is to do as they are told to, which involves executing build jobs dispatched by the Master.
- You can configure a project to always run on a particular Slave machine, or a particular type of Slave machine, or simply let Jenkins pick the next available Slave.
Let us see how to configure slave nodes on Ubuntu EC2. If you like to learn how to setup Jenkins Master on Ubuntu EC2 instance,

[﻿click here](https://www.coachdevops.com/2023/03/install-jenkins-on-ubuntu-2204-setup.html) 

.

Jenkins Master uses SSH keys to communicate with slave. You need to create ssh keys in Jenkins master by executing below command.

**Pre-requisites:**

- [﻿Jenkins Master is already setup and running](https://www.coachdevops.com/2023/03/install-jenkins-on-ubuntu-2204-setup.html) 
- Create a new EC2 instance for Slave
**Steps involved:**

1. Setup new EC2 instance for slave

2. Create jenkins user and Install Java, Maven in Slave node

3. Create SSH keys and upload public keys from master to slave node.

4. verify ssh connection from master to slave

5. Register slave node in Jenkins master

6. Run build jobs in Jenkins slave

### 

### Slave node configuration
(You need to create at least t2.

small Ubuntu 22.0.4 instance

 for this slave)

**only port 22 needs to be open**

**Change Host Name to Slave**

`sudo hostnamectl set-hostname Slave` 

**Install Java**

`sudo apt update` 

`sudo apt install openjdk-17-jdk -y` 

**Install Maven**

`sudo apt install maven -y` 

**Create User as Jenkins**

`sudo useradd -m jenkins` 

`sudo -u jenkins mkdir /home/jenkins/.ssh` 

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj11aZaLBkkNg2zkSip8H4XDm7bx0L4GVE6Y3j3bhT7XkWDoIEEjVWRFHaVZEzGKypopeMKrUEokIcEkmX8L0wi6eqrQLvSTirdRY636_4MXzXT7sejeuUB_nmI6bZ_HpaZubjWMGPXvC6s/s320/Screen+Shot+2018-10-20+at+3.55.18+PM.png "")

#### 

#### **Now Login to Jenkins Master**
Create SSH keys by executing below command:

`ssh-keygen` -t rsa -m PEM

Please over write your existing keys.

**Copy SSH Keys from Master to Slave **

Execute the below command in Jenkins master EC2.

`sudo cat ~/.ssh/id_rsa.pub` 

Copy the output of the above command:

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgfae8KUglYWoyg11TCmhbyevj8frFRWbbl7Z5uwHum1_5Zf5bOXuwjw99-MKDHvYtyYFDuGK5e8BUgAs6gqS9tW8z7u-ZiPHEMIcx6mn1Y41le_XjhkseJRKWAgS9NYvXKu9s5CE887bLL/s320/Screen+Shot+2019-01-09+at+8.35.52+PM.png "")

**Now Login to Slave node and execute the below command**

`sudo -u jenkins vi /home/jenkins/.ssh/authorized_keys` 

This will be empty file, now copy the public keys from master into above file.
Once you pasted the public keys in the above file in Slave, come out of the file by entering :wq!


### **Now go into master node**
`ssh jenkins@_**slave_node_ip**_`


![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi-yZ9L03gE8JQn_BINR_a6UpEmZHqyhdIeXqtwR0xlLdGcDyViEupC594Zr92TEpRHaRy0ZBV2GCyMfPTxr2Sx4fe6SYyVPy1CbvrKyxTjHFTFPndPh1x6SFdv7yPYcpSj6GQh2PdqYn-I/s320/Screen+Shot+2018-10-20+at+4.00.18+PM.png "")




this is to make sure master is able to connect slave node. once you are successfully logged into slave, type exit to come out of slave.

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgzjAODbIQzPxkm3R4FjJ_p_UVlAgTq5aIxicaLtPnm9pevjF8VbrHSWg9OQ-5gxm_vzYxTi7qT1nRQbkX3nrXMZ3W1f-G7n2uCZk28vI6gFwPHktCCH1DKOPh1kKt8a6QcHvZXyMSNkuoY/s320/Screen+Shot+2018-10-20+at+4.01.31+PM.png "")

### 

### 

### **Register slave node in Jenkins:**
Now to go Jenkins Master, manage jenkins, manage nodes.


![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhNh0fkDz5Rh170V4XnzZsoFmK7bw2UZdWAyUHw2hqw166wiq8d2YARqBzH64Rva9LO_L1hLAy-TIacWVwPOUwr9PK6mnWlVLHQkdjjsLZMWf7UHZ2x5De9EKQ3qyRnupzSBUJUm7qMdRqb/s320/Screen+Shot+2018-10-20+at+4.03.40+PM.png "")









Click on new node. give name and check permanent agent.
give name and no of executors as 1. enter `/home/jenkins` as remote directory.
select launch method as Launch slaves nodes via SSH.
enter Slave node ip address as Host.

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiZY7cxzx7qjR45tbdICtQUY4P9uAIhBkP5Ad791csoZ0cYlvUfchezCz8TbwNytqdZWj3061j4zabWxac7lWjCAXBsMpDknFBO4IPREl07tLNNldB4s8k4TIXSC2fYpjb2IfeT_6IBDqNM/s320/Screen+Shot+2018-10-20+at+4.08.31+PM.png "")











click on credentials. Enter user name as jenkins. Make jenkins as lowercase as it is shown.
 Kind as SSH username with private key. enter private key of master node directly by executing below command:

![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiO3HauUo4EJ5QVMFtqm2Yr56SvO394tvHlednyTCQqFoLcBVcyrOorS9_iu8OdWhmt26mDbZgcMLRbAk41Ajt0pakJhFWr8NbFCqxtyDfLyF6OIriw4nksqxjgCkEmbLIyXg8vbieR9Hxo/s320/Screen+Shot+2018-10-20+at+4.11.41+PM.png "")

`sudo cat ~/.ssh/id_rsa`
(Make sure you copy the whole key including the below without missing anything)
-----BEGIN RSA PRIVATE KEY-----
-----
-----END RSA PRIVATE KEY-----

click Save.
select Host key verification strategy as "manually trusted key verification strategy".

Click Save.
Click on launch agent..make sure it connects to agent node.


![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgQLJKl2MEjUbD4eoOYL5C-wwWfElrxhO8_cp0azzmCdunBnwqQyesOxAOwarG14O4mFuFq48U8GYWsVqjEQnRnlIa5c_yN28KCTZ9ZHK0K7nCavHfWQIgn6iJbGJV432IT59xYdC2535Px/s320/Screen+Shot+2018-10-20+at+4.13.45+PM.png "")


Now you can kick start building the jobs, you will see Jenkins master runs jobs in slave nodes.

