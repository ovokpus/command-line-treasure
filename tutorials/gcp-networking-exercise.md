# GCP Networking Exercise

---

## Instructions

1. Create a network called database-network that has a single subnet called database-a and an IP range of 10.1.0.0/24 located in europe-north1

2. For security reasons, you are requested not to publicly expose the MySQL resources (that listen on port 3306 TCP) inside the database-network, which means that it needs to be private and not reachable from the Internet.

3. The Webserver Network Allows SSH and HTTP Traffic. Your next task also relates to security. webserver-network contains web server VMs, so you are required to allow HTTP (port 80) traffic from all the Internet. Your manager also requires you to allow SSH (port 22) traffic from the webmaster computer which has a public IP of 152.89.1.10. This way he can manually manage VMs inside the network. There should only be these two firewall rules associated with the webserver-network.

4. The Database Network Allows MySQL Traffic. You also need to provide security features for the database-network. You are required to allow MySQL (port 3306) traffic only from the webserver-network, so make sure to allow traffic coming from the two subnets into the webserver-network.

5. The VPC Peering Has Been Set Up. Because the VMs inside the webserver-network need to communicate with the DB in the database-network, you need to provide a VPC peering to let resources inside different networks communicate.

6. Cloud NAT associated with Cloud Router. Your manager also tells you that he wants the DB cluster to be able to communicate to the internet for downloading the last updates/releases of the DBMS. The last task you need to perform is to create a cloud NAT gateway with a cloud router named database-network-router which is located in europe-north1.

---

### Step-by-step Guide

#### **1. Create the Networks and Subnets:**

- **webserver-network**:
  - Navigate to the VPC networks section in the Google Cloud Console.
  - Click 'Create VPC network'.
  - Name: `webserver-network`.
  - Add two subnets:
    - **webserver-a**:
      - IP range: `10.0.0.0/24`
      - Region: `us-central1`
    - **webserver-b**:
      - IP range: `10.0.2.0/24`
      - Region: `us-east1`
  - Click 'Create'.

- **database-network**:
  - Repeat the steps above but name it `database-network`.
  - Add one subnet:
    - **database-a**:
      - IP range: `10.1.0.0/24`
      - Region: `europe-north1`

---

#### **2. Secure the database-network:**

- Navigate to the 'Routes' section.
- Find the route associated with `database-network` that has a destination range of `0.0.0.0/0` and delete it. This ensures that there's no public internet access by default.

---

#### **3. Set Up Firewall Rules for webserver-network**

- Go to the 'Firewall' section in the Cloud Console.
- Click 'Create firewall rule':
  - Name: `allow-http`.
  - Network: `webserver-network`.
  - Targets: `All instances in the network`.
  - Source IP ranges: `0.0.0.0/0`.
  - Specified protocols and ports: tcp:80
  - Click 'Create'.

- Repeat the above process:
  - Name: `allow-ssh-from-webmaster`.
  - Source IP ranges: `152.89.1.10/32`.
  - Specified protocols and ports: tcp:22

---

#### **4. Set Up Firewall Rule for database-network**

- Create a new firewall rule:
  - Name: `allow-mysql`.
  - Network: `database-network`.
  - Targets: `All instances in the network`.
  - Source IP ranges: `10.0.0.0/24,10.0.2.0/24`.
  - Specified protocols and ports: tcp:3306

---

#### **5. Set Up VPC Peering**

- Navigate to 'VPC network peering' section.
- Click 'Create connection':
  - Name: `webserver-to-database`.
  - VPC network: `webserver-network`.
  - Peered VPC network: `database-network`.
- Click 'Create'.

- Create another:
  - Name: `database-to-webserver`.
  - VPC network: `database-network`.
  - Peered VPC network: `webserver-network`.

---

#### **6. Create Cloud NAT with Cloud Router**

- Navigate to 'Cloud Routers' under 'Hybrid Connectivity'.
- Click 'Create Router':
  - Name: `database-network-router`.
  - Region: `europe-north1`.
  - Network: `database-network`.

- Navigate to 'Cloud NAT'.
- Create a NAT gateway:
  - Name: `nat-for-database`.
  - Router: `database-network-router`.
  - Region: `europe-north1`.
  - Choose either an existing IP or allow Google Cloud to auto-allocate IPs.

---
