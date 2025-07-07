## Lab-5: Configuring Docker Machine as Jenkins Slave, build and deploy code in Docker Host as a container

**Objective:**
In this lab, you will set up a Docker container as a Jenkins slave, build a Docker image for a Java web application, and deploy it in a Docker container.


### Task-0: Configuring Key pair (for Jenkins to SSH into Docker Server).

### Step 1: SSH into the Jenkins Server

From your **jumpserver or local terminal**, SSH into the Jenkins server:

```bash
ssh -i <your-key.pem> ubuntu@<jenkins-server-public-ip>
```

Switch to the `root` user:

```bash
sudo su
```

---

### Step 2: Generate SSH Key Pair on Jenkins Server (as root)

```bash
ssh-keygen
```

* Press `ENTER` at all prompts (to accept default file location and no passphrase).
* This creates:

  * Private key: `/root/.ssh/id_rsa`
  * Public key: `/root/.ssh/id_rsa.pub`

---

### Step 3: Copy the Public Key

Display the public key:

```bash
cat /root/.ssh/id_rsa.pub
```

Copy the full output (it will start with `ssh-rsa`...).

---

### Step 4: SSH into Docker Server

From your terminal (new tab or logout from Jenkins), SSH into Docker server:

```bash
ssh -i <your-key.pem> ubuntu@<docker-server-public-ip>
```

Switch to root:

```bash
sudo su
```

---

### Step 5: Add Jenkins Public Key to Docker's Authorized Keys

Ensure `.ssh` directory exists:

```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
```

Edit the authorized keys file:

```bash
vi /root/.ssh/authorized_keys
```

* Paste the **Jenkins public key** you copied earlier.
* Save and exit: press `ESC`, then type `:wq!` and press `ENTER`.

Set correct permissions:

```bash
chmod 600 /root/.ssh/authorized_keys
```

---
### Step 6: Test SSH from Jenkins to Docker

Go back to your **Jenkins server**, as `root`, and run:

```bash
ssh root@<docker-server-private-ip>
```

* You should log in **without password**.
* If successful, SSH key setup is complete.

---


### Task-1: Configuring Docker Machine as Jenkins Slave.

1. Go to **Jenkin's home page** and click on the **Manage Jenkins** and **Nodes**.
2. Click on **New Node** in the next window. Give the node name as **docker-slave** and Select **"permanent agent"**
3. Fill out the details for the node docker-slave as given below.
* The name should be given as **docker-slave**,

The Remote root directory is the folder on the Docker server (slave node) where Jenkins will:

* Store workspace files
* Execute build jobs
* Manage agent runtime
## If you're using root user to connect, You must create it on the Docker server if it doesn’t exist:
```
sudo mkdir -p /root/jenkins
sudo chown root:root /root/jenkins
```
* Remote Root Directory as **/root/jenkins**,
* labels to be **Slave-Nodes**,
* usage to be given as **"use this node as much as possible"**
* Launch method to be set as **"launch agents via SSH"**.
* In the host section, give the **Private IP of the Docker instance**.
* For Credentials for this Docker node, click on the dropdown button named **Add** and then click on **Jenkins**;
* Then in the next window, in kind select **SSH username with private key** (give username as root),
* In **Private Key** Select **Enter directly** then Copy-Paste the Private Key from the jenkins server and then click on **Add** .

**Note:**
To get the private key, Go to your **Jenkins-server** and run below command.
```
cd /root/.ssh
cat id_rsa
```
* Copy the entire content, including the **first and last lines**. Paste it into the space provided for the **private key** then click on **Add**.
* Now, In SSH Credentials, choose the newly created **root** credentials.
* Host Key Verification Strategy Select as **"Non Verifying Verification Strategy"** and **Save** it.
* Click on the **Add** button.
* it's done.
---

### Task-2: Build and deploy code in Docker Host on the container.

#### Now In CLI, SSH into your Docker Host and Perform the below steps to create a "Dockerfile" in /home/ubuntu directory.
```
cd /home/ubuntu
```
```
vi Dockerfile
```
enter the below:
```
# Pull Base Image
FROM tomcat:8-jre8

# Maintainer
MAINTAINER "CloudThat"

# Copy the war file to the images tomcat path
ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/
```
---------------------------------------------------------------------
#### Explanatory Notes of Above Docker FIle:

The Dockerfile you provided is a basic Dockerfile for deploying a Java web application using the Tomcat web server as the base image. Here's what each part of the Dockerfile does:

1. `FROM tomcat:8-jre8`: This line specifies the base image for your Docker container. In this case, it uses the official Tomcat 8 image with the Java 8 runtime. This image contains the Tomcat web server and Java runtime environment.

2. `MAINTAINER "CloudThat"`: This line is a comment and specifies the maintainer or author of the Dockerfile.

3. `ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/`: This line adds your Java web application's WAR file (`hello-world-war-1.0.0.war`) to the `/usr/local/tomcat/webapps/` directory inside the container. When Tomcat starts, it will automatically deploy the WAR file as a web application.

This Dockerfile is a simple example of how to package a Java web application in a Docker container. It uses an existing Tomcat image as the base, and you're adding your application's WAR file to it. You can further customize this Dockerfile based on your specific requirements, such as adding environment variables, configuring Tomcat, or specifying additional settings for your application.

Once you build this Docker image and run a container based on it, your Java web application should be accessible through Tomcat on port 8080 inside the container, as mentioned in your Jenkins job's post-build steps.

---------------------------------------------------------------------
1. Go to your **Jenkins Home page**, click on the **drop-down** on **hello-world project**, and select Configure 
tab.
2. In **General Tab**, Select **Restrict where this project can be run** and enter Label Expression as 
**Slave-Nodes.**
3. Go to **Post Steps Tab**, select **"Run only if the build succeeds"** then click on **Add post-build** step select **Execute shell** from the drop-down and copy paste the below commands in the shell and **Save**

**Execute shell commands in Jenkins:**
#### Note: You may replace 'yourname' with your actual first name (lines 3 and 5).

```
#!/bin/bash

# Navigate to the Jenkins workspace
cd /home/ubuntu/workspace/hello-world/

# Copy the WAR file to the root directory
cp -f target/hello-world-war-1.0.0.war .

# Stop and remove the existing Docker container (if it exists)
sudo docker stop helloworld-container || true
sudo docker rm helloworld-container || true

# Build the Docker image using the Dockerfile in the /home/ubuntu directory
sudo docker build -t helloworld-image -f /home/ubuntu/Dockerfile .

# Run the Docker container
sudo docker run -d -p 8080:8080 --name helloworld-container helloworld-image

```
---------------------------------------------------------------------
#### Explanatory Notes of the above Commands:

The commands you provided are part of the Jenkins job's post-build steps, and they are responsible for building a Docker image and running a Docker container for your Java web application. Here's a breakdown of what each command does:

1. `cd /home/ubuntu/workspace/hello-world/`: Change the working directory to the Jenkin's workspace

2. `cp -f target/hello-world-war-1.0.0.war .`: Copy the WAR file (presumably the artifact of your Java web application) from the Jenkins workspace to the root directory (`~`), where you'll perform the Docker build.

3.`sudo docker stop helloworld-container || true`:  Stop any existing Docker container with the name "helloworld-container" forcefully if it exists. 

4. `sudo docker rm helloworld-container || true`: Remove any existing Docker container with the name "helloworld-container" forcefully if it exists. 

5. `sudo docker build -t helloworld-image -f /home/ubuntu/Dockerfile .`: Build a Docker image with the tag "helloworld-image" based on the Dockerfile located in the current directory (`.`). The Dockerfile you created earlier specifies how the image should be built.

6. `sudo docker run -d -p 8080:8080 --name helloworld-container helloworld-image`: Run a Docker container named "helloworld-container" from the "helloworld-image" image. This container will be detached (`-d`) and will map port 8080 on the host to port 8080 inside the container.

After running these commands, your Java web application should be deployed inside a Docker container, and it will be accessible on port 8080 of your Docker host.

Make sure that Docker is properly configured on your host, and all the necessary dependencies for your Java web application are included in the WAR file. Additionally, ensure that the Dockerfile is correctly configured to set up your application inside the container.

---------------------------------------------------------------------
* Now you can build your **hello-world project** Either by
1. Clicking on **"Build Now"** or
2. By **making a small change in Github files**.

* Once the build is successful, access the tomcat server page,

To access the Page In Browser Type **"http:// < Your Docker Host Public IP >:8080/hello-world-war-1.0.0/"** to see the website
* **Example:** http://3.95.192.77:8080/hello-world-war-1.0.0/
---
**Summary:**
1. Configure a Jenkins slave node ie... named as "docker-slave."
2. Configure the slave node to use SSH for communication.
3. Set up a Dockerfile to define the Docker image for the Java web application.
4. Create a Jenkins job to build and deploy the Java web application in a Docker container.
5. Add post-build steps to the Jenkins job to copy the WAR file, build the Docker image, and run a Docker container.
6. Trigger the Jenkins job to build and deploy the application in the Docker container.
7. Access the deployed web application in the Docker container via the Docker host's public IP.

---------------------------------------------------------------------

#### =============================END of LAB-05=============================
---
