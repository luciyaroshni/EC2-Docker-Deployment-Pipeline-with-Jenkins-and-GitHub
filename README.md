# EC2-Docker-Deployment-Pipeline-with-Jenkins-and-GitHub
In this project, I am building a Jenkins CI/CD pipeline with GitHub webhook integration to deploy a Docker application on EC2 instances using a declarative pipeline. 

In this project, I am building a Jenkins CI/CD pipeline with GitHub webhook integration to deploy a Docker application on EC2 instances using a declarative pipeline. This documentation captures my learning journey and insights gained from implementing this project. Thanks to Chetan Rakhra for sharing this idea.

Here’s a summary of the key actions:

1. AWS Setup: Set up an EC2 instance with Ubuntu as the AMI and open necessary ports for HTTP, HTTPS, Docker, and Jenkins.
2. EC2 Configuration: Connect to the instance, generate SSH keys, install Jenkins and Docker, and set up firewall rules for required ports. Jenkins Setup: Configure Jenkins by accessing its dashboard, installing plugins, creating an admin user, and verifying installations of Jenkins and Docker.
3. CI/CD Pipeline Creation: Create a Jenkins pipeline to pull code from GitHub, build a Docker image, and deploy a container on EC2. Jenkins will automate the deployment process using GitHub webhook triggers.
4. GitHub Webhook Integration: Enable webhook integration in GitHub, triggering Jenkins to rebuild and redeploy the Docker container with every code update pushed to the repository.
5. Automation Validation: After configuring the project with GitHub webhooks and testing, any new code changes in GitHub will automatically initiate the pipeline in Jenkins, ensuring up-to-date deployments.

### Step 1: EC2 creation

I first created an EC2 instance, which serves as the deployment environment for running the Dockerized application once the Jenkins pipeline deploys it.

I navigated to the EC2 service and clicked on “Launch an instance,” where I provided the instance name, selected the AMI (Ubuntu), and chose the instance type (t2.micro).

![image](https://github.com/user-attachments/assets/dc61984c-d213-41f4-a05a-1352b2a7658b)

![image](https://github.com/user-attachments/assets/d3b78062-0d23-4584-aa74-cb587c62f6a8)

![image](https://github.com/user-attachments/assets/2c1d52da-6590-45f3-b880-6a9e36336aa8)

I clicked on “Create new key pair” and provided a name for the key pair.

![image](https://github.com/user-attachments/assets/8fd6bc92-d413-4031-95b4-a32a47566d02)

Under Network settings, checked the boxes for allow HTTP and HTTPS.

![image](https://github.com/user-attachments/assets/0dc42790-0624-43cf-9721-0caf908d2dfc)

Clicked on Launch instance.

![image](https://github.com/user-attachments/assets/cd759a3d-436b-41b0-a453-fade0f0111a3)

Instance successfully created.

### Step 2: EC2 Configuration

I copied the SSH from the server, navigated to the folder where I downloaded the .pem file (key pair), and opened a terminal from that location.Then past the SSH.

I got an error <l>“Permissions 0644 for ‘docker.pem’ are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.”</l>

![image](https://github.com/user-attachments/assets/4db3a971-32a8-4235-948e-981914e94c9c)

It happened because the permissions on my private key file (docker.pem) were too open, which made SSH refuse to use it for security reasons.

I resolved the error by modifying the permissions to ensure the owner has read access. I ran the following command:

```
chmod 400 docker.pem
```

Next, I generated SSH key pairs, enabling secure communication between the EC2 instance and other services without requiring additional passwords. I created a public key to share with external services for recognition and a private key that remains on the EC2 instance for identity verification. This setup simplifies automatic connections, particularly for tasks such as the CI/CD pipeline.

To generate the public and private keys, run the below command:

```
ssh-keygen
```

![image](https://github.com/user-attachments/assets/12b17ef2-514c-492f-b3e2-f3af8abcfc32)

Next, I installed Jenkins on the EC2 instance. The installation requires Java, which must be set up prior to installing Jenkins.

For installing java, run the below command:

```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
```

For installing Jenkins:

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

Check the status of Jenkins by,

```
sudo service jenkins status
```

![image](https://github.com/user-attachments/assets/0d3f6561-90fb-47f0-8edd-4ffe6e4eb067)

Next, I needed to install Docker. Docker is crucial for enabling Jenkins to build and deploy your application.

```
sudo apt install docker.io
```

I installed Docker successfully.

Now I can focus on setting up firewall rules for required ports. I will allow ports 8080 and 8001 for the machine.

To do this, I returned to the AWS console and navigated to the EC2 service. I selected my EC2 instance from the list and scrolled down to the security section at the bottom.

Then I clicked on the security group.

![image](https://github.com/user-attachments/assets/a9c065ff-dbcf-414a-956c-ffa9a2d30873)

Click on Edit inbound rules.

![image](https://github.com/user-attachments/assets/46d6d4fb-e35f-451f-a9fe-ec2e80a6f821)

Clicked on the Add rule. Selected custom TCP, and for port, provide 8080 and 8001.

![image](https://github.com/user-attachments/assets/d5c435d9-e8f0-475b-8147-38544092a183)

Saved the rule and accessed the Jenkins by accessing the public IP with port 8080.

I successfully accessed it, and it prompted me for the password, which I retrieved from the specified path. After entering the password, I was asked to install plugins and set up credentials.

![image](https://github.com/user-attachments/assets/0e6835fa-ca20-4453-984e-7142c5567ab5)

![image](https://github.com/user-attachments/assets/bcc726e4-44f7-43e9-8122-8f57d844292d)

Successfully set the Jenkins.

![image](https://github.com/user-attachments/assets/e0098bf6-d631-4398-a520-b745c64b9631)

### Step 3: CI/CD Pipeline Creation

The next step involved creating a CI/CD pipeline to fetch the code from GitHub.

For this, I accessed the Jenkins dashboard and clicked on “New Item.”

