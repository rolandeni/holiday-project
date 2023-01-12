<<<<<<< HEAD
# Holiday Challenge

![vpc1](images/chrismas.jpg)
* ### To Create private instances with no public ip address
* ### To create a VPC,route tables and subnets for those instances
* ### To create a load balancer for the instances.
* ### To install Nginx on the server and configure Nginx for PHP files.
## Requirements
+ internet connection
+ AWS account 
+ Basic knowlege of instance.
+ Basic knowlege of Ansible. 


### To carry out these tasks i will breakdown the process into simple steps.
# step 1
1. Create a VPC: on AWS
on the search panel input VPC, then click on create VPC. now select VPC and more to make the process much easier (AWS helps you create all other resource need for VPC).
+ input a name tag eg. projectVPC.
![vpc1](images/vpc.webp)
+ For IPV4 CIDR block you can put a range of ip address that will be given to your instances in that VPC eg. 10.0.0.0/16 

![vpc1](images/vpc2.webp)
+ NAT gateway in AZ (Availability Zone) select 1 in AZ this will create a single gateway for internet connection to your instance.

![vpc1](images/vpc3.webp)
+ select S3 Gateway
+ Enable DNS hostname
+ Enable DNS resolution
+ click create VPC 

![vpc1](images/vpc4.webp)

**Note:**  Any option not mentioned leave it as default.

# step 2 
2. Create EC2 instance: on AWS search panel input EC2. Now click on launch instance

![vpc1](images/ec2.webp)
+ input a name for your instance. eg. server
+ on the summry pannel, Number of instance = 3

![vpc1](images/ec22.webp)
+ for os image select ubuntu or any image you are familiar with. this is the operating system that will run on the servers.

**Note:** for this demo practice i will be  selecting free tier version of all the resource in you.

![vpc1](images/ec23.jpg)
+ create a key pair that we would use through out this demo session. eg. key1
+ key pair type RSA , key format .pem then create key
![vpc1](images/ec24.jpg)
+ now select the created key pair.
+ click network settings to view all hidden options.

    + VPC - select the previously created VPC i.e projectVPC
    ![vpc1](images/networkset.jpg)
    + Subnet - select a private subnet
    + auto assign public ip - Disable
    + firewall(security group) select create security group.
        + input security group name. eg. instanceSECURITY
        + Edit the inbound rule to look like below.
        ![vpc1](images/secrule1.jpg)
        + leave outbound rule as defaut.
    + select the created security group
    + click launch instance.

# step 3
3. To create a bastion Host: this is also an instance but its purpose is to help communicate with the private servers created in the private subnet. the bastion host will be created in same VPC but a public subnet.
+ On AWS search panel input EC2, input name bastion ,select operating system ubuntu.
+ select private key previously created i.e key1
+ click the network settings to view the hidden options
+ select the previously created VPC, select a public subnet.
+ select Auto asign public ip address - enabled
+ select the previous security group. i.e instanceSECURITY
+ now launch instance.
+ connect to the instance using ssh client( with private key on CLI) or aws instance connect( without private key on browser).
+ while connected to the bastion host create these fies.
    + sudo nano ansible.yml, then paste these template.
```       
---

- hosts: all
  become: yes
  tasks:

  - name: update & upgrade server
    apt:
      update_cache: yes
      upgrade: yes

  - name: install nginx
    apt:
      name: nginx
      state: latest

  - name: install php to run our code
    apt:
      name: php-fpm
      state: latest

  - name: remove the nginx file
    file:
      path: /var/www/html/index.nginx-debian.html
      state: absent

  - name: remove the default nginx file
    file:
      path: /etc/nginx/sites-available/default
      state: absent

  - name: copy the nginx default configuration file to server
    copy:
      src: default
      dest: /etc/nginx/sites-available
      owner: root
      group: root
      mode: 0744

  - name: To copy the php web page file to server.
    copy:
      src: index.php
      dest: /var/www/html
      owner: root
      group: root
      mode: 0744

  - name: restart nginx
    service:
      name: nginx
      state: restarted
      enabled: yes

```
+ save and close the file.
+ create another file called **default** *sudo nano default*
+ paste the below code 
```

[defaults]
inventory = inventory
private_key_file = ~/root-server1.pem
```
+ save and close the file create another file called **ansible.cfg**
+ *sudo nano ansible.cfg*  
+ paste this text then save and close file.
```
[defaults]
inventory = inventory
private_key_file = ~/key1.pem
```
+ create the last file that is your private key *sudo nano key1.pem*
+ open your downloaded private key, copy it ,then paste it and save with same name from AWS.
+ now run *sudo chmod 400 key1.pem* this is to change the permission of the private key so it will not be rejected.
+ create another file called **inventory** *sudo nano inventory* 
+ paste the private ip address of the 3 servers you previously created.
+ now create another file called **index.php**
*sudo nano index.php* then paste this code. 
```
<HTML>
<head>







```
+ now run this code so we can make an insatllation 

*sudo apt update; sudo apt dist-upgrade -y*
+ now install Ansible
 
 *sudo apt install ansible*

**Note**: all the above actions is carried out on bastion server. To confirm that your bastion server can communicate with the private servers run this code on the bastion host 

*ssh -i "key pair name" ubuntu@private ip address* 

i.e ssh -i key1.pem ubuntu@10.0.0.23

+ now run the ansible.yml playbook using the command *ansible-playbook -i inventory filename.yml* 
i.e ansible-playbook -i inventory ansible.yml
![vpc1](images/playbook2.jpg)

**NOTE:** these five files must be present on the bastion host before you run the ansible script.
*1.ansible.yml 2.default 3.inventory 4.index.php 5.key1.pem*

# step 4 
4. To create load balancer and Target group
+ on the AWS search panel input EC2 
+ scroll down to target group option
+ select create target group.
![vpc1](images/target.webp)
+ select instances
+ input a target group name eg. target1
![vpc1](images/target2.jpg)
+ protocol HTTP port -80
+ VPC select the previously created VPC.
+ select HTTP1 , then next page
![vpc1](images/target3.jpg)
+ tick the radio buttons for the private servers created
+ click *include as pending below*
![vpc1](images/target3.webp)
+ click on the part that says load balancer *none associated*
![vpc1](images/target4.webp)

# step 5
5. creating a load balancer. for this practice we will be using Application load Balancer.
+ select create Application load balancer
+ inpute a name eg. loadB
+ select internet facing and ipv4.
+ select the previously created vpc.
+ under mapping, select all the public subnets buttons.
+ security group - create security group 
  + input security group name eg loadSEC
  + select the previosly created VPC.
  + create four inbound rules.
    + custom tcp - port 80 - anywhere ipv4.
    + custom tcp - port 80 - anywhere ipv6.
    + custom tcp - port 443 - anywhere ipv4.
    + custom tcp - port 443 - anywhere ipv6.
+ outbound rule all traffic 
+ save and create security group
+ select the new security group ie. loadSEC
+ listener HTTP -protocol -HTTP -port 80- default action select target group previously created.
+ click create load balancer.

# step 6
6. to set instance security group to face lod balancer.
+ on AWS console search EC2 
+ scroll down to security group
+ select the security group created for instance.
+ add a new rule.
+ custon tcp - port 80 - then load balancer security group.
+ click save info .

# step 7
7. To test the load balancer DNS link and connect it to route 53. 
+ click load balancer, then copy the DNS link and paste on your browser. it should open the newly created PHP page. 
+ on AWS console search panel inpute route 53 
+ create a hosted zone
+ edit the hosted zone record
+ turn on *Alias* then select load balancer from the options
+ then inpute load balancer DNS.
+ save the record
+ now connect any domain name you have with the hosted zone created.



=======
# Holiday-project
## Using ansible to install and configure Nginx.
## Created 3 servers on aws and placed them behind a Load Balancer.
## Create a private VPC and then made the instance private without having a public IP.
>>>>>>> 76fcf5e33e0b3e257039d14c7b0dd932fa0ec37c

