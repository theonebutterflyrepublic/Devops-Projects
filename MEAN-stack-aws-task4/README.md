MEAN STACK DEPLOYMENT TO UBUNTU IN AWS

# Book Register MEAN Stack Project

## 1. Project Overview
The **Book Register** is a simple web application built using the **MEAN stack**:

- **MongoDB** – Document database (Stores book records (name, ISBN, author, pages)).  
- **Express** – Backend application framework (Handles API routes between client and database).  
- **AngularJS** – Frontend framework for dynamic views and client-side interaction.  
- **Node.js** – Runtime environment for executing JavaScript on the server.  

This project demonstrates a **full-stack web application** deployed on an **Ubuntu EC2 instance**

## 2. Prerequisites

- AWS account and EC2 instance (Ubuntu 20.04 LTS).  
- Node.js  
- MongoDB 
- npm installed.  
- Basic understanding of terminal/SSH commands.

## 3. Project Setup
   ### Step 0- Launch an ec2 instance on aws and ssh

   ### Step 1 – Update system and install Node.js
 using

 ```bash
 sudo apt update
 ```

```bash
 sudo apt upgrade
```

#add certificate
```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
```


![images](images/001-install%20certificate.jpg)

NOTE: The Certificate command installs important system tools that help your server download and verify software securely and it is often run before setup scripts.


# run this command
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
```

NOTE: This command:

Downloads a setup script from NodeSource, Runs it as administrator, Adds Node.js 20 repository to Ubuntu, Prepares your system to install Node.js 20. So after running it, you can install Node.js 20

![images](images/002.jpg)

# next install nodejs to the latest version. 
run
```bash
sudo apt install -y nodejs
```
![images](images/003-installnjs.jpg)

verify installation

# run

```bash
nodejs -v
npm -v
```




