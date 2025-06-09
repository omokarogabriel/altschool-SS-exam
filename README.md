## **Cloud Engineering Second Semester Examination Project**

_This Exam uses the Modern Approach (AWS)_

## **Setting up the aws infrastructure**

**List of resources used for the infrastructure**

- **vpc** (For Creating the isolated network within the cloud, it uses 10.0.0.0/16 cidr block and has total of 256 networks and each networks will have total IP of 254).
- **Igw** (For allowing access to and fro within the internet to the vpc, and it is attached to the VPC).
- **route-table and route** (For routing or for direction of traffic arrange the network).
- **subnet** (For breaking down networks into smaller segment, and the cidr block is 10.0.1.0/24, whereby this network has total of 254 IP addresses, it is linked with the route table).
- **security group** (Virtual firwall that allows traffic from an ec2 instance to another be it port 80, port 22(ssh) or port 443, it is stateful).
- **key-name** (For ssh access, either from a specified IP or from the internet).
- **ec2 instance with public IP enable** (For the server).

## **Installing the packages and setting up the server**

_I have successfully ssh into my ec2_

```bash
ssh -i AwsKey ubuntu@52.87.162.241
```

_Instaling the packages_

```bash
sudo apt update # this update the package library
apt install apache2
apt enable apache2
apt start apache2

# to check if apache2 is running
systemctl status apache2
# the aws has handled the firwall for us

# navigate into the directory
cd /var/www/html
git clone https://github.com/omokarogabriel/portfolio.git

```

<!-- paste the ip address in the browser -->

_52.87.162.241_

<!-- image of my web page -->

```html
<img src="./exam.png" alt="Diagram" width="400" />
```
