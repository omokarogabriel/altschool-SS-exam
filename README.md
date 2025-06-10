## **Cloud Engineering Second Semester Examination Project**

_This Exam uses the Modern Approach (AWS)_

## **Setting up the aws infrastructure**

**List of resources used for the infrastructure**

- **vpc** (For Creating the isolated network within the cloud, it uses 10.0.0.0/16 cidr block and has total of 256 networks and each networks will have total IP of 254).

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications 'ResourceType=vpc, Tags=[{Key=Name, Value=MyVPC}]'
```

- **Igw** (For allowing access to and fro within the internet to the vpc, and it is attached to the VPC).

```bash
aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway, Tags=[{Key=Name, Value=MyIGW}]'
```

- **route-table and route** (For routing or for direction of traffic arrange the network).

```bash
aws ec2 create-route-table --vpc-id vpc-00976cf179c37d4fc --tag-specification 'ResourceType=route-table, Tags=[{Key=Name, Value=MyPublicRouteTable}]'

aws ec2 create-route --route-table-id rtb-02aec2da260da42ce --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0d29e295df140798f
```

- **subnet** (For breaking down networks into smaller segment, and the cidr block is 10.0.1.0/24, whereby this network has total of 254 IP addresses, it is linked with the route table).

```bash
aws ec2 create-subnet --vpc-id vpc-00976cf179c37d4fc --cidr-block 10.0.1.0/24 --region us-east-1 --tag-specification

aws ec2 attach-internet-gateway --internet-gateway-id igw-0d29e295df140798f --vpc-id vpc-00976cf179c37d4fc

aws ec2 associate-route-table --route-table-id rtb-02aec2da260da42ce --subnet-id subnet-0130bb2b3a6b76ab1 --region us-east-1
```

- **security group** (Virtual firwall that allows traffic from an ec2 instance to another be it port 80, port 22(ssh) or port 443, it is stateful).

```bash
aws ec2 create-security-group --group-name examSG --description "security group for ec2 instances" --vpc-id vpc-00976cf179c37d4fc

# ssh
aws ec2 authorize-security-group-ingress --group-id sg-0aa2c7b52d1dff232 --protocol tcp --port 22 --cidr 0.0.0.0/0

# http
aws ec2 authorize-security-group-ingress --group-id sg-0aa2c7b52d1dff232 --protocol tcp --port 80 --cidr 0.0.0.0/0
```

- **key-name** (For ssh access, either from a specified IP or from the internet).
*my key name is "AwsKey"*

- **ec2 instance with public IP enable** (For the server).

```bash
aws ec2 run-instances --image-id ami-0fc5d935ebf8bc3bc --instance-type t2.micro --key-name AwsKey --subnet-id subnet-0130bb2b3a6b76ab1 --security-group-ids sg-0aa2c7b52d1dff232 --associate-public-ip-address --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyServer}]' 
```

<!-- ## **Installing the packages and setting up the server**

_I have successfully ssh into my ec2_

```bash
ssh -i AwsKey ubuntu@52.90.76.179
```

_Instaling the packages_

```bash

# Update package library
sudo apt update

# Install Apache2 web server
sudo apt install apache2 -y

# Enable Apache2 to start on boot
sudo systemctl enable apache2

# Start Apache2 service
sudo systemctl start apache2

# Check Apache2 service status
sudo systemctl status apache2


# the aws has handled the firwall for us, so we dont need to install or enable ufw

# navigate into the directory
cd /var/www/html
git clone https://github.com/omokarogabriel/portfolio.git

``` -->


## **Using Ansible to install packages and deploy the git repo into the ec2 server**

## **The inventory**
**It takes in the IP address for the host, and ssh into the ec2 instance using the private key**
```yml
webservers:
  hosts:
    52.90.76.179:
      ansible_user: ubuntu
      ansible_ssh_private_key_file: /home/omokaro/AwsKey
```

## **The webserver role and default varaibles**

**The default variables**
```yml
---
# defaults file for webserver
# This file contains default variables for the webserver role.
# You can override these variables in your playbook or inventory files.
webserver_packages:
  - apache2
  - git

apache_service_name: apache2
apache_service: 
  name: "{{ apache_service_name }}"
  state: started
  enabled: true  
```

**The webserver role task**
```yml
---
# tasks file for webserver
- name: Update apt cache
  apt:
    update_cache: yes
  when: ansible_os_family == "Debian"

  # Ensure the apt cache is updated before installing packages
  # This task is only run on Debian-based systems
  # such as Ubuntu, which is common for web servers.
  # This task is idempotent, meaning it will not run if the cache is already up to date.

- name: Install web server & packages
  apt:
    name: "{{ item }}"
    state: present
    update_cache: yes
  with_items: "{{ webserver_packages }}"
  loop:
    - "{{ webserver_packages }}"

- name: Ensure Apache is running
  service:
    name: "{{ apache_service_name }}"
```


## **The git repo role and default variables**

**The default variables**
```yml
---
# defaults file for git-repo
my_git_repo: 'https://github.com/omokarogabriel/portfolio.git'
my_git_dest: '/var/www/html/portfolio'
my_git_version: 'main'
my_git_update: yes
my_git_force: yes
my_new_dest: '/var/www/html'
```

**The git repo role task**
```yml
---
# tasks file for git-repo
- name: Ensure git is installed
  apt:
    name: git
    state: present
  become: true


- name: Clone or update web application repo
  git:
    repo: '{{ my_git_repo }}'
    dest: '{{ my_git_dest }}'
    version: '{{ my_git_version }}'
    update: '{{ my_git_update }}'
    force: '{{ my_git_force }}'

- name: Ensure the web application directory and contents have correct permissions
  file:
    path: '{{ my_git_dest }}'
    state: directory
    recurse: yes
    mode: '0755'
    owner: ubuntu
    group: ubuntu
  become: true

- name: Move contents from portfolio to html
  shell: mv {{ my_git_dest }}/* {{ my_new_dest }}
  args:
    removes: '{{ my_git_dest }}'
  become: true


- name: Remove empty portfolio directory
  file:
    path: '{{ my_git_dest }}'
    state: absent
  become: true
```


## **The playbook**
**It runs all the tasks in the role**
```yml
---
- name: Apply webserver role on webservers and deploy git repository
  hosts: webservers
  become: yes
  roles:
    - webserver
    - git-repo
```



### **Below is the repo to the Ansible file**
*[ansible file repo](https://github.com/omokarogabriel/my-first-static-inv-ansible)*

### **Below is the github repo link**
*[github repo](https://github.com/omokarogabriel/portfolio)*

<!-- paste the ip address in the browser -->
*The IP address where the web page is hosted*
- 52.90.76.179

<!-- image of my web page -->

<!-- ```html
<img src="./exam.png" alt="Diagram" width="400" />
``` -->
**HOME PAGE**
![MyHome](./home.png)

<!-- **ABOUT PAGE**
![MyImage](./about.png) -->

**PROJECT PAGE**
![MyProjects](./projects.png)

**SERVICE PAGE**
![MyServices](./services.png)

**CONTACT PAGE**
![MyContact](./contact.png)
