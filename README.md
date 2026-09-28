# GitHub Actions + SonarQube + Docker + EKS Project
## 1. Create and Clone the GitHub Repository
Create an empty repository in GitHub with the name:
**Github-Actions-Project**
Open VS Code → Terminal → Git Bash and clone the repository:
```bash
git clone https://github.com/harathi-mutyam/Github-Actions-Project.git
```
## 2. Push Local Code to GitHub (optional here)
**Open Git Bash in the project directory.**
```bash
# Remove the existing Git configuration:

rm -rf .git
# Initialize Git:

git init

# Rename the branch to main:

git branch -M main

# Add the .gitignore file:

git add .gitignore

# Commit the changes:

git commit -m "Add gitignore to project"

# Add all project files:

git add .

# Commit the project:

git commit -m "project"

# Add the GitHub remote repository:

git remote add origin https://github.com/harathi-mutyam/Github-Actions-Project.git

# Push the code:

git push -f origin main
```
## 3. Create the AWS Security Group

**Open the AWS Console → EC2 → Security Groups.**
```bash
#Create a security group named:

githubaction_sg


 ```
####  Configure the required inbound rules:
######  Security Group Inbound Rules

| Type        | Protocol | Port       | Source          |
|-------------|----------|------------|-----------------|
| SMTPS       | TCP      | 465        | Anywhere IPv4   |
| Custom TCP  | TCP      | 587        | Anywhere IPv4   |
| Custom TCP  | TCP      | 3000–11000 | Anywhere IPv4   |
| HTTP        | TCP      | 80         | Anywhere IPv4   |
| HTTPS       | TCP      | 443        | Anywhere IPv4   |
| SSH         | TCP      | 22         | My IP           |

## 4. Create the GitHub Actions Self-Hosted Runner

**Create an EC2 instance with the following configuration:**

```bash
•	Name: Runner 
•	OS: Ubuntu 
•	Instance type: c7i-flex.large 
•	Key pair: github-key 
•	Security Group: githubaction_sg 
•	Storage: 25 GB
```
**Connect to the instance using Git Bash:**
```bash
ssh -i Downloads/github-key.pem ubuntu@<public-ip-of-runner-ec2-instance>

# Update the system:

sudo apt update

# Set the hostname:

sudo hostnamectl set-hostname runner
/bin/bash

```
## 5. Configure the GitHub Self-Hosted Runner

**Open your GitHub repository:**

**Actions → Runners → Self Hosted Runner New self-hosted runner → Linux**
```bash
Copy the commands displayed by GitHub and execute them on the Runner EC2 instance.
```
```bash
For reference:

copy those commandsopen Runner gitbash-->paste those commands and run itAgain go back to git hub repository in browser copy the download the latest runner package command--> paste it in Runner gitbash and run it -->again open in browser github respository copy the extract the installer command--> paste it in the Runner gitbash and run it

After downloading and extracting the runner package:
```
```bash
cd actions-runner

# Check the files:

ls

If required, remove the downloaded tar file:

rm actions-runner-linux-x64-2.322.0.tar.gz

ls
```
```bash

For Reference:

Now Configure the Runner: open github repository in browser copy the command related to create Runner & start the Configuration experience paste it in EC2 Runner Gitbash & run it

Press Enter for Default here type: Press Enter-->Enter-->Type Runner1 -->Provide Labels:  here type self-hosted 
Enter the name of work folder: Press enter key here for default
```

**Configure the runner using the command provided by GitHub.**
```bash
When prompted:
•	Press Enter for the default values where appropriate. 
•	Enter Runner1 as the runner name. 
•	Enter self-hosted as the label. 
•	Press Enter to use the default work folder.

ls

```
```bash
for reference: Open github repository in browseropen same repository in duplicate browser--> select settings -->actionsrunners--> Now you can see Runner1 is in offline 

```
**open EC2 runner gitbash  --> run the below command**
```bash

# Start the runner:

./run.sh

```
**You should see a message similar to:**
```bash
Connected to GitHub

Listening for Jobs

The runner is now connected to GitHub and ready to execute GitHub Actions jobs.

```
For referece: Open github repository in browser Refresh the page -->You can observe status changed to **idle or active**

## 6. Install Maven on the Runner

Check the cicd.yaml file in:

**.github/workflows/cicd.yaml**

```bash
Since the workflow uses Maven, Maven must be installed on the Runner EC2 instance.
```
**Open one more gitbash for installation purpose.**


Connect to the Runner:


```bash
ssh -i Downloads/github-key.pem ubuntu@<public-ip-of-runner-ec2-instance>

sudo hostnamectl set-hostname runnerinstalationpurpose

/bin/bash

Install Maven:

sudo apt install maven -y
```
**Go to GitHub → Actions and run the workflow again.(click on re-run jobs)**

At this stage,**the pipeline should proceed successfully through the compile job phase.**

## 7. Create the SonarQube EC2 Instance

Create another EC2 instance for SonarQube.

```bash
Configuration:
•	Name: SonarQube 
•	OS: Ubuntu 
•	Instance type: c7i-flex.large 
•	Key pair: github-key (old one)
•	Security Group: githubaction_sg (old one) 
•	Storage: 25 GB
```

**Connect to the SonarQube instance:**
```bash
ssh -i Downloads/github-key.pem ubuntu@<public-ip-of-sonarqube-ec2-instance>

# Set the hostname:

sudo hostnamectl set-hostname sonarqube

/bin/bash

# Update the system:

sudo apt update

# Install Docker: type

docker  #Install docker through sonarqube

sudo apt install docker.io -y

```
**Add the current user to the Docker group:**
```bash
sudo usermod -aG docker $USER

# Apply the group changes:

newgrp docker

# Run SonarQube:

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```
## 8. Open SonarQube
Open a browser and enter:
```bash
http://<sonarqube-ec2-public-ip>:9000
```
**Use the default login credentials:**
```bash
Username: admin
Password: admin
```
**After login, the Change Password page will appear.**
```bash
Enter:
Old Password: admin
New Password: admin123
Confirm Password: admin123

Click Update Password.
```
**You will then be redirected to the SonarQube dashboard.**

## 9. Generate the SonarQube Token
**In SonarQube:**
```bash
Administration(top navigation bar) → Security → Users
 
Under the token section click on this symbol  .It will open this page 
 
Using the above one generate a new token.

Provide a name for the token (Name : A) and click Generate.

Copy the generated token and store it securely. 

```
## 10. Add SonarQube Secrets to GitHub

**Open:**
**Our project GitHub Repository in browser → Settings → under Security and quality Secrets and variables → Actions-select secrets tabnew repository secret button**
```bash
Create a repository secret:

Name: SONAR_TOKEN (The name must be match the name used in cicd.yaml file under build_project_and_sonar_scan: job)

Value: paste the <SonarQube-token> here

The name must match the name used in cicd.yaml.


Create a repository variable:

Now select variable tab --> new Repository Variable-->

Name: SONAR_HOST_URL

Value: http://<sonarqube-ec2-public-ip>:9000


The variable name must also match the name used in cicd.yaml
```
## 11. Install Docker on the GitHub Actions Runner

**If the pipeline fails during the Docker stage, install Docker on the Runner EC2 instance( for that open runnerinstalationpurpose gitbash).**

Follow the official Docker installation instructions for Ubuntu.

For reference: open google in browser search docker install in ubuntuclick the official link  https://docs.docker.com/engine/install/ubuntu/ -->

copy the Set up Docker's apt repository commands from browser , **paste and run in runnerinstalationpurpose gitbash termina**

# Add Docker's official GPG key:
```bash
sudo apt update

sudo apt install ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```
# Add the repository to Apt sources:
```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update

```
**copy Install the Docker packages commands from the official docker website --> run those commands in runnerinstalationpurpose gitbash terminal**
```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# After installing Docker, add the Ubuntu user to the Docker group:

sudo usermod -aG docker ubuntu

# Apply the changes:

newgrp docker
```
**for reference: close the both runner related gitbashes: runnerinstalationpurpose gitbash and runner gitbash also.**

Reconnect to the Runner and start the GitHub Actions runner again:
```bash
ssh -i Downloads/github-key.pem ubuntu@<public-ip of runner ec2-instance>

sudo hostnamectl set-hostname runner

/bin/bash

ls

cd actions-runner

./run.sh

```
Your original notes use the official Docker Ubuntu website for installation process.

## 12. Configure Docker Hub
**Open the cicd.yaml file and update the Docker image name with your Docker Hub username and application name.**
```bash
# Example:

tags: harathi2026/bankapp:latest
```
**Create a Dcoker username and password in our github repository as secrets and variable:**

Open again our project github repository page in browser-->settings-->under Security and quality-->select Secrets and variables-->select Actions-->select secrets Tab-->new repository secret button
```bash
Name: DOCKERHUB_TOKEN (name must be match in cicd.yaml file under build_docker_image_and_push: job)

Secret: Enter Your Docker password Click on Add secret button

```
**select variables Tab-->new repository variable button**
```bash
select variables Tab-->new repository variable button

Name: DOCKERHUB_USERNAME (name must be match in cicd.yaml file under build_docker_image_and_push: job)

Value: Enter Your Docker username -->Click on Add Variable button
```
**Create the following GitHub repository secret:**
```bash

DOCKERHUB_TOKEN

Store your Docker Hub password/token as the secret value.

Create the following repository variable:

DOCKERHUB_USERNAME

Store your Docker Hub username as the value.
```
**Commit the changes and run the GitHub Actions workflow again.**
```bash

For reference:

Open cicd.yaml file -->commit the changes--> or click on Actions -->  rerun the job 
```

## 13. Create the EC2 Server for EKS

**Create another EC2 instance to create the EKS cluster.**
```bash
Configuration:

•	Name: Server 
•	OS: Ubuntu 
•	Instance type: c7i-flex.large 
•	Key pair: github-key 
•	Security Group: githubaction_sg 
•	Storage: 25 GB

```
**Connect to the server: open one more gitbash terminal**
```bash

ssh -i Downloads/github-key.pem ubuntu@<public-ip-of-server-ec2-instance>

# Update the system:

sudo apt update

# Set the hostname:

sudo hostnamectl set-hostname server

/bin/bash
```
## 14. Install unzip , AWS CLI

**Install the required packages:**
```bash
sudo apt update

sudo apt install unzip -y

# Download AWS CLI:

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

# Extract it:

unzip awscliv2.zip

# Install AWS CLI:

sudo ./aws/install
 ```
**For reference:**
**open aws console in browsergenerate Access key and secret key for IAM user or for Root user**

click on Root user (rightside top corner)-->security credentials-->select create a access key-->select the check box I understand-->select create access key button--> **copy or download the secret and access keys**

```bash
# Configure AWS:

aws configure

# Provide:

AWS Access Key ID: paste <your-access-key> here

AWS Secret Access Key: paste <your-secret-key> here

Default region name: eu-north-1

Default output format: json
```
## 15. Install Terraform

**Install the required packages:**
```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl

# Add the HashiCorp GPG key:

wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

# Add the HashiCorp repository:

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

# Install Terraform:

sudo apt-get update && sudo apt-get install terraform

# Verify the installation:

terraform --version
```
**These are the Terraform installation steps documented .if required check it from official documentation also. optional notes check it down**
```bash 
# 1. Install prerequisites

sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl

# 2. Add HashiCorp GPG key
wget -O- https://apt.releases.hashicorp.com/gpg |     gpg --dearmor |     sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
# 3. Add the repository
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" |     sudo tee /etc/apt/sources.list.d/hashicorp.list
# 4. Update and install
sudo apt-get update && sudo apt-get install terraform
# 5. Verify installation
terraform –version

 ```



## 16. Clone the EKS Terraform Code

**The EKS Terraform code is also stored in the GitHub Actions project repository.**

Before cloning the project, open variables.tf and update the SSH key name:
```bash

variable "ssh_key_name" {
  description = "The name of the SSH key pair to use for instances"
  type        = string
  default     = "github-key"
}

``` 
**Replace github-key with the key pair name you are using in ec2 instances(runner,sonarqube,server instances).**

Also check main.tf and update the AWS region if required:

```bash 
provider "aws" {
  region = "eu-north-1"
}
``` 
Clone the repository in **server instance gitbash terminal**:

```bash

git clone https://github.com/harathi-mutyam/Github-Actions-Project.git

# Move into the project directory:

cd Github-Actions-Project

# Initialize Terraform:

terraform init

# Create the infrastructure:

terraform apply --auto-approve
``` 
EKS deployment producing cluster, node group, subnet, and VPC outputs. 

## 17 resources must be created
Copy the 
```bash 
cluster_id = "eks-cluster"
node_group_id = "eks-cluster:eks-node-group"
subnet_ids = [
  "subnet-0f697578532397c9a",
  "subnet-00efb5a12c4ec3159",
]
vpc_id = "vpc-0c7afca4f7cfd81cd"
``` 
## 17. Install kubectl in server Gitbash terminal
```bash 
# Download kubectl:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

# Install it:
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify:

kubectl version –client
 ``` 
## 18. Configure kubectl for EKS

**Update the kubeconfig:**

aws eks update-kubeconfig --region eu-north-1 --name <your-eks-cluster-name>

```bash 
aws eks update-kubeconfig --region eu-north-1 --name eks-cluster
# Verify the worker nodes:

kubectl get nodes

ls

# View the kubeconfig:

cat ~/.kube/config
``` 
**The kubeconfig content is then used for the GitHub Actions deployment configuration.**

## 19. Add EKS Secrets to GitHub

Go to:
```bash 
GitHub Repository → Settings → Secrets and variables → Actions → Secrets
``` 
**Create a Repository secret:**
```bash

Name: EKS-KUBECONFIG

Value: <entire ~/.kube/config content>

Also create:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

Store the corresponding AWS credentials as GitHub Actions secrets.
```

For reference: Save the aws access key and secret key as secrets in our project github repository
 ```bash

Select secrets tab-->select new repository secret button

name: AWS_ACCESS_KEY_ID

secret : paste the aws access key value from aws account  here -->save it

Select secrets tab-->select new repository secret button


name: AWS_SECRET_ACCESS_KEY

secret : paste the aws secret key value from aws account here -->save it
```
**Then go to Actions and run the pipeline again.**

## 20. Install AWS CLI and kubectl on the Runner

If the GitHub Actions runner reports that AWS CLI is missing, install it on the Runner EC2 instance-->**open runnerinstalationpurpose gitbash terminal.**
```bash
cd actions-runner

Check AWS CLI:

aws --version

# Install AWS CLI if required:

sudo apt update

sudo apt install -y unzip curl

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

# Verify:

aws --version
```

**Install kubectl:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify:

kubectl version --client
```
**Run the GitHub Actions pipeline again from the github repository.**

## 21. Verify the Application

#### After the GitHub Actions pipeline completes successfully, 

**open the Server Git Bash terminal.**

Run:
```bash
kubectl get all
```
**Find the application's DNS name from the output.**

##### Open the DNS name in a browser and verify that the application is accessible.
```bash
Then:
1.	Register a new user. 
2.	Create a username and password. 
3.	Log in using the new credentials. 
4.	Verify that the application is working correctly.
 ```
## 22. Cleanup

After verifying the application, destroy the Terraform-created resources from the Server(open server girtbash terminal):
```bash
terraform destroy --auto-approve

17 resources must be destroyed
```
Then:
**•	Terminate the remaining EC2 instances(runner instance, sonarqube instance, server instance) from the AWS Console.**


**•	Disable and delete the AWS access key after it is no longer required.** 
```bash
Project Flow
GitHub Repository
       ↓
GitHub Actions
       ↓
Self-Hosted Runner
       ↓
Maven Build
       ↓
SonarQube Code Analysis
       ↓
Docker Build
       ↓
Docker Hub
       ↓
Terraform
       ↓
AWS EKS Cluster
       ↓
kubectl
       ↓
Kubernetes Application
       ↓
Browser
```
```bash
Note: Our cicd.yaml file already contains the required configuration.
In a real-time project, if you need any additional actions, you can search for them in the GitHub Actions Marketplace, such as SonarQube Quality Gate Check or any other action required for your project, and add the appropriate action to the cicd.yaml file.

After making the changes, commit the updated cicd.yaml file and run the GitHub Actions pipeline again. At this stage, observe the pipeline execution.
If the pipeline fails, check the error message. In our case, the Docker stage failed, so we need to troubleshoot the Docker-related issue.
```

