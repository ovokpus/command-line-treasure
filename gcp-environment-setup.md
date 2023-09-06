# Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab

## Challenge Scenario

As a cloud engineer in Jooli Inc. and recently trained with Google Cloud and Kubernetes you have been asked to help a new team (Griffin) set up their environment. The team has asked for your help and has done some work, but needs you to complete the work.
You are expected to have the skills and knowledge for these tasks so don’t expect step-by-step guides.
You need to complete the following tasks:

* Create a development VPC with three subnets manually
* Create a production VPC with three subnets manually
* Create a bastion that is connected to both VPCs
* Create a development Cloud SQL Instance and connect and prepare the WordPress environment
* Create a Kubernetes cluster in the development VPC for WordPress
* Prepare the Kubernetes cluster for the WordPress environment
* Create a WordPress deployment using the supplied configuration
* Enable monitoring of the cluster via stackdriver
* Provide access for an additional engineer

### Some Jooli Inc. standards you should follow:

* Create all resources in the us-east1 region and us-east1-b zone, unless otherwise directed.
* Use the project VPCs.
* Naming is normally team-resource, e.g. an instance could be named kraken-webserver1.
* Allocate cost effective resource sizes. Projects are monitored and excessive resource use will result in the containing project's termination (and possibly yours), so beware.
* This is the guidance the monitoring team is willing to share: unless directed, use e2-medium.

---

### TASK 1: Create development VPC manually

Create a VPC called griffin-dev-vpc with the following subnets only:

1. griffin-dev-wp; IP address block: 192.168.16.0/20
2. griffin-dev-mgmt; IP address block: 192.168.32.0/20

```bash
gcloud compute networks create griffin-dev-vpc --subnet-mode custom

gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region us-east1 --range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region us-east1 --range=192.168.32.0/20
```

---

### TASK 2: Create production VPC manually

Create a VPC called griffin-prod-vpc with the following subnets only:

1. griffin-prod-wp; IP address block: 192.168.48.0/20
2. griffin-prod-mgmt; IP address block: 192.168.64.0/20

```bash
gsutil cp -r gs://cloud-training/gsp321/dm .

cd dm

sed -i s/SET_REGION/us-east1/g prod-network.yaml

gcloud deployment-manager deployments create prod-network \
 --config=prod-network.yaml

 cd ..
```

---

### TASK 3: Create bastion host

Create a bastion host with two network interfaces, one connected to griffin-dev-mgmt and the other connected to griffin-prod-mgmt. Make sure you can SSH to the host.

```bash
gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b

gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc

gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc
```

---

### TASK 4: Create and configure Cloud SQL Instance

1. Create a MySQL Cloud SQL Instance called griffin-dev-db in us-east1.
2. Connect to the instance and run the following SQL commands to prepare the WordPress environment:

```bash
gcloud sql instances create griffin-dev-db --root-password password --region=us-east1

gcloud sql connect griffin-dev-db

```

from the sql prompt:

```sql
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
exit;
```

---

### TASK 5: Create Kubernetes cluster

Create a 2 node cluster (e2-standard-4) called griffin-dev, in the griffin-dev-wp subnet, and in zone us-east1-b.

```bash
gcloud container clusters create griffin-dev \
 --network griffin-dev-vpc \
 --subnetwork griffin-dev-wp \
 --machine-type n1-standard-4 \
 --num-nodes 2 \
 --zone us-east1-b

gcloud container clusters get-credentials griffin-dev --zone us-east1-b

cd ~/

gsutil cp -r gs://cloud-training/gsp321/wp-k8s
```

---

### TASK 6, 7, 8

1. Use Cloud Shell and copy all files from gs://cloud-training/gsp321/wp-k8s.
   The WordPress server needs to access the MySQL database using the username and password you created in task 4.
2. You do this by setting the values as secrets. WordPress also needs to store its working files outside the container, so you need to create a volume.
3. Add the following secrets and volume to the cluster using wp-env.yaml.
4. Make sure you configure the username to wp_user and password to stormwind_rules before creating the configuration.
   You also need to provide a key for a service account that was already set up. This service account provides access to the database for a sidecar container.
5. Use the command below to create the key, and then add the key to the Kubernetes environment:

```bash
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .

cd wp-k8s

sed -i s/username_goes_here/wp_user/g wp-env.yaml

sed -i s/password_goes_here/stormwind_rules/g wp-env.yaml

kubectl create -f wp-env.yaml

gcloud iam service-accounts keys create key.json --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

kubectl create secret generic cloudsql-instance-credentials --from-file key.json
----------------------------------------------------------------------------------
I=$(gcloud sql instances describe griffin-dev-db --format="value(connectionName)")

sed -i s/YOUR_SQL_INSTANCE/$I/g wp-deployment.yaml

kubectl create -f wp-deployment.yaml

kubectl create -f wp-service.yaml
```

---

### TASK 8: Enable monitoring

1. Navigation Menu -> Kubernetes Engine -> Services and Ingress -> Copy Endpoint's address.
2. Navigation Menu -> Monitoring -> Uptime Checks -> + CREATE UPTIME CHECK Title : Wordpress Uptime
3. Next -> Target Hostname : {Endpoint's address} (without http...) Path : / // Next -> Next -> Create

---

### TASK 9: Provide access for an additional engineer

1. Navigation Menu -> IAM & Admin -> IAM -> ADD New Member : {Username 2 from Lab instruction page} Role : Project -> Editor
2. Save.
