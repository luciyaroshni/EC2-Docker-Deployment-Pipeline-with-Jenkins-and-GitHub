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

![image](https://github.com/user-attachments/assets/e048a73e-dabd-4a95-b84b-e7aff34786fd)

It asked for a name, and I provided the name as todo-app.

Selected the Item type as Freestyle project.

Then clicked on Ok.

![image](https://github.com/user-attachments/assets/e2450255-805c-48ca-bcdd-2b881cd9fabd)

Provided the description.

![image](https://github.com/user-attachments/assets/68f38e3e-da1a-4add-8f62-da8e3d56c81c)

In the Source Code Management section, I selected Git and needed to add the Repository URL and credentials.

For the credentials, I needed to create a credential on GitHub. I went to Settings, then Personal access token, and generated a new token.

![image](https://github.com/user-attachments/assets/c7dc9b07-bd3e-499b-a66e-53bb7d0c3a56)

I provided a note and selected the necessary scopes before clicking on “Generate token.”

![image](https://github.com/user-attachments/assets/c0278bf2-8211-41e0-9209-6ba43d0bb9b2)

I added that token as a password on the credentials

![image](https://github.com/user-attachments/assets/ff208a62-4f54-4fcd-8086-1e5d202a036e)

In the Build Triggers section, I configured the GitHub hook trigger for Git SCM polling, enabling Jenkins to automatically trigger builds when it receives a webhook from GitHub.

![image](https://github.com/user-attachments/assets/9b0f40ea-dea9-4cd6-9bd2-ee5ab64839ba)

In the Build Steps, I defined the actions Jenkins should take after fetching the code, such as building a Docker image. To do this, I clicked on “Add build step” and selected “Execute shell.”

I provided the below commands for building and deploying Docker.

```
docker build -t todo-app .
docker run -p 8001:8001 -d todo-app
```

docker build -t todo-app . command will instruct Docker to build a new image using the Dockerfile located in the current directory.

docker run -p 8001:8001 -d todo-app command will runs a new container from the todo-app image. The -p 8001:8001 option maps port 8001 of your host (the EC2 instance) to port 8001 of the container. This allows you to access the application running inside the container from outside the container on your EC2 instance’s public IP at port 8001.

![image](https://github.com/user-attachments/assets/a8b16d76-2892-4dd0-930f-9a74c59ad1f1)

After providing the command, I clicked on save.

Then, ran Build Now.

![image](https://github.com/user-attachments/assets/8028480d-37da-4018-8809-dbfae19842ef)

After clicking “Build Now,” I encountered an error. I suspected it was related to my branch name, and it turned out I was correct.

![image](https://github.com/user-attachments/assets/715c180c-31f1-4fb1-b143-269c7a96e884)

I provided the branch name as master when in the repository it is mentioned as main.

![image](https://github.com/user-attachments/assets/e6376c3d-77e7-4fa7-9baf-a706e213b913)

I modified it and built it again.

Okay… and I had several errors at this point :|

Now, I received an error indicating that there was a “permission denied” issue while trying to connect to the Docker daemon on the EC2 instance.

![image](https://github.com/user-attachments/assets/3949afc9-ee06-4e61-bc48-a0d649cc0040)

Jenkins accessing Docker, which is controlled locally on the EC2 instance by user permissions. Adding Jenkins to the Docker group allowed it to use Docker without needing root privileges.

```
sudo usermod -aG docker jenkins
```

After that, I restarted Jenkins to ensure the permissions took effect.

```
sudo systemctl restart jenkins
```

I ran the build again, and finally, it succeeded.

![image](https://github.com/user-attachments/assets/6a8b8e85-4c37-4594-8a1d-e72114d9c2dc)

I tried to access the application, but it wasn’t loading. It seemed there were some dependency issues that caused the Docker container to exit.

I made some changes to my application’s code and added an additional dependency in the requirements.txt, which brought the Docker container back up.

![image](https://github.com/user-attachments/assets/93953e47-7170-47d1-9327-e453851dbad6)

But still, the URL was not loading in the browser.

The issue was that the code specified port 80, so I changed it to 8001 and attempted to build again. However, the build failed again because a container was already running. I had to stop the running container before starting the build process again.

Now it was successfully up, and I could access my application also.

![image](https://github.com/user-attachments/assets/117e162d-0037-45b3-9843-419eb8dca610)

### Step 4: GitHub Webhook Integration

To set up GitHub Webhook Integration for Jenkins, I configured my GitHub repository to send a webhook to Jenkins whenever there’s a push event. This triggered Jenkins to rebuild and redeploy my Docker container automatically.

I set up SSH key authentication for secure communication between my server and GitHub.

I navigated to Settings in GitHub, then clicked on SSH and GPG keys.

![image](https://github.com/user-attachments/assets/591c7f29-7980-4a0c-a0f2-092bbdd132df)

Clicked on Add SSH key

![image](https://github.com/user-attachments/assets/20f58799-8efd-4b38-9f32-e2f5730a13b0)

I successfully added the key to my GitHub.

Next, I wanted to create a webhook in my GitHub repository.

I navigated to the specific repository, accessed Settings, and then clicked on Webhooks. I clicked on Add webhook.

![image](https://github.com/user-attachments/assets/128718c1-a88f-4c30-889f-962b6d10d6f7)

I specified the Payload URL, which is the URL of my Jenkins server that will handle the incoming webhook requests. I also set the Content-Type to application/json, which is the format for the data sent from GitHub to Jenkins.

Clicked on Add webhook

![image](https://github.com/user-attachments/assets/cf71518c-2293-4aaf-8ea2-fc0c65346edd)

It was successfully created.

![image](https://github.com/user-attachments/assets/f3a59d1a-8469-468c-a896-08bf563697ef)

However, I encountered a problem. After making changes, the code was automatically pushed to Jenkins. During the building process, it failed because the old container was still running and occupying the same port.


![image](https://github.com/user-attachments/assets/02f86f98-8474-4a44-b45b-324e9859c3f7)

So I modified the Jenkin pipeline to handle the old containers.

![image](https://github.com/user-attachments/assets/5a80baf0-331a-49f6-9c4f-9895f2ea264e)

Since the container name was dynamically changing, I specified a fixed name in the script, ensuring that the container would always be created with this designated name.

After modifying the code, the build ran successfully.

![image](https://github.com/user-attachments/assets/dd115fdd-4071-4ac5-ae26-bd10681d1881)

As you can see, I created it successfully. I have implemented a CI/CD pipeline that integrates Jenkins with GitHub for automated deployment of a Docker application on an EC2 instance.



