# Setting Up a Self-Hosted GitHub Actions Runner on AWS EC2 with CI&CD Pipeline

## Introduction

This repository shows steps to set up a self-hosted GitHub Actions runner on an AWS EC2 instance. This setup allows you to run GitHub Actions workflows on your own hardware, providing more control over the environment and potentially reducing costs. Moreover once the runner is setup, your application will be deploy automatically from the created workflow files which have defined the whole end to end CI&CD pipeline and nginx proxy server on behalf of you on that AWS EC2 machine.


## Prerequisites

1. **AWS Account:** You need an AWS account to create an EC2 instance.
2. **GitHub Account:** You need a GitHub repository where you have admin access.
3. **Basic Knowledge:** Familiarity with GitHub Actions, AWS EC2, and basic Linux commands.


## Steps to deploy application

### 1. Launch an EC2 Instance

1. **Sign in to the AWS Management Console.**
2. **Navigate to the EC2 Dashboard.**
3. **Click "Launch Instance" to create a new instance.**
4. **Add Tags:** Add tags to help manage your instance (e.g., git-workflow).
5. **Choose an Amazon Machine Image (AMI):** Select a Linux-based AMI, such as Ubuntu (64-bit x86).
6. **Choose an Instance Type:** Select an instance type based on your needs (e.g., t2.micro or t3.micro for testing).
7. **Generate Key pair:** Create new key pair with key_pair_type:RSA and file_format:.pem to connect to host if needed.
8. **Configure Network and Security Group:** Add rules to allow inbound traffic (e.g., SSH (port 22) for remote access).

> [!TIP]
> For testing purpose create new security group and allow all trafic (Allow SSH traffic, Allow HTTPS traffic, Allow HTTP traffic). You can modify them later from attached security group.

9. **Add Storage:** Configure storage if the default is sufficient (8GB to 12 GB is sufficient).
10. **Review and Launch:** Review your settings and launch the instance. Make sure to create or select a key pair for SSH access.

    ![AWS EC2 Instance Config](https://github.com/user-attachments/assets/ff7a7d5f-5578-4344-a70b-a6380f252479)

### 2. Connect to Your EC2 Instance

1. **Open a terminal on your local machine.**
2. **Use SSH to connect to your EC2 instance:**

   ```bash
   $ ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
   ```
> [!NOTE]
> Connect to AWS EC2 instance through AWS Management Console


### 3. Setup Self Hosted Runner

1. **Open your repository:** Go to Settings --> Actions --> Runners --> New self-hosted runner. You will see steps to deploy your self hosted runner macOS, Linux or Windows. Choose Linux, you will see similler code to setup your runner:

    **Download**
    ```bash
    # Create a folder
    $ mkdir actions-runner && cd actions-runner# Download the latest runner package

    $ curl -o actions-runner-linux-x64-2.319.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.319.1/actions-runner-linux-x64-2.319.1.tar.gz# Optional: Validate the hash

    $ echo "3f6efb7488a183e291fc2c62876e14c9ee732864173734facc85a1bfb1744464  actions-runner-linux-x64-2.319.1.tar.gz" | shasum -a 256 -c# Extract the installer

    $ tar xzf ./actions-runner-linux-x64-2.319.1.tar.gz
    ```

    **Configure**
    ```bash
    # Create the runner and start the configuration experience
    $ ./config.sh --url https://github.com/Roni-Boiz/cicd-ec2-runner --token <your-token>

    # Last step, run it!
    $ ./run.sh
    ```
    
    ![Runner Setup](https://github.com/user-attachments/assets/04106930-975f-4563-88fe-00e3af72d1d1)

2. **Execute each command one after the other:** Go to your EC2 instance and execute each and every code snippets. When you enter `./config.sh` enter follwoing details:
    - runner group --> keep as default
    - name of runner --> git-workflow
    - runner labels --> git-workflow
    - work folder --> keep default

> [!TIP]
> At the end you should see **Connected to GitHub** message upon successful connection

![Runner Success](https://github.com/user-attachments/assets/42d6a34a-7cad-4eb9-8bc7-64822d8c2b82)

### 4. Start Build and Deply Pipeline

Once the runner is configured, pipeline will start automatically on connected EC2 instance and deploy your aplication. 

You can see the progress of each workflow in the actions tab. Upon completed of nginx workflow you should be able to access your application in EC2 instance **public DNS address**.

![Application](https://github.com/user-attachments/assets/d27395a2-179a-4e62-bb77-75d353ffcf5c)

### 5. Verify the Pipeline

Make change to you application and see will it reflect on the next build.

**Docker Workflow**

![Docker Workflow](https://github.com/user-attachments/assets/d3c16f1c-37af-4773-b1e7-88926c544c5d)

**CI&CD Workflow**

![CI&CD Workflow](https://github.com/user-attachments/assets/7052666a-386c-4c0c-81df-58402629bf94)

**Nginx Workflow**

![Nginx Workflow](https://github.com/user-attachments/assets/c76ee008-5868-4a05-b448-062a8096b546)

> [!TIP]
> If every thing is working as intend you can update the docker workflow to stop run it each time you make changes to main branch and leave it as manual workflow by commenting out `on --> push --> branch --> main`

### 6. Remove Self Hosted Runner

Finally if you are done or want to stop and remove your application and terminate the instance.

1. **Open your repository:** Go to Settings --> Actions --> Runners --> Select your runner (git-workflow) --> Remove Runner.Then you will see steps safely remove runner from EC2 instance.

    ![Remove Runner](https://github.com/user-attachments/assets/76f1c354-fa4c-46c4-adf8-8475030894ba)

2. **Remove runner:** Go to your EC2 instance and execute the command

    ```bash
    // Remove the runner
    $ ./config.sh remove --token <your-token>
    ```
    
> [!WARNING]
> Make sure you are in the right folder `~/actions-runner`

![Remove Runner](https://github.com/user-attachments/assets/092e190f-84ea-4135-bd69-f1e05ebb19ca)

2. **Terminate Instance:** Go to your AWS Management console --> EC2 terminate the created instance (git-workflow) and then remove any additional resources (vpc, security groups, etc)

    **Verify that every resource is removed or terminated**
