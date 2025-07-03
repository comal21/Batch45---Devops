## Lab 3: Configuring Jenkins server and Installing Tomcat onto Jenkin's Server for Deploying our Application.

**Objective:**
The objective of this lab is to configure Jenkins to build and deploy applications. It includes `Setting up Jenkins,` `installing necessary plugins` and `configuring Jenkins to build Maven projects,` and `Installing Tomcat Server.` 

---------------------------------------------------------------------
### Task-1: Configure Jenkins Server:


Get the **Initial Password** for Jenkins from the below path.
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
   (**Example:** "afbe8d33e25b4b908c0b9f91546f09e6")

1. Now, go to the **Web Browser** and enter the Jenkins URL as shown: **http://< Jenkin's Public IP>:8080/**
2. Under Unlock Jenkins, enter the above **initialAdminPassword** & click **Continue**.
3. Click on **Install suggested Plugins** on the Customize Jenkins page.
4. Once the plugins are installed, it gives you the page where you can create a New **Admin User**. 
5. Enter the **User Id** and **Password** followed by **Name and E-Mail ID** then click on **Save & Continue**.
6. In the next step, on the Instance Configuration Page, verify your **Jenkins Public IP** and **Port Number** then click on **Save and Finish**

#### Now you will be prompted to the Jenkins Home Page

1. Click on **Manage Jenkins** > **Plugins** > **Available Plugins** tab and search for **Maven**.
2. Select the "**Maven Integration**" Plugin and "**Unleash Maven**" Plugin and click **Install** (without restart).
3. Once the installation is completed, click on **Go back to the top page**.
4. On Home Page select **Manage Jenkins** > **Tool**.
5. Inside Tool Configuration, look for **Maven installations**, click **Add Maven**. 
6. Give the **Name as "Maven"**, choose **Version as 3.9.5**, and **Save** the configuration.
7. Now you need to make a project for your application build, that selects **New Item** from the Home Page of Jenkins
8. Enter an item name as **hello-world** and select the project as **Maven Project** and then **click OK.**
   ( You will be prompted to the configure page inside the hello-world project.)
9. Go to the "**Source Code Management**" tab, and select Source Code Management as **Git**, Now you need to provide the GitHub Repository **Master Branch URL** and **GitHub Account Credentials**.
10. In the Credentials field, you have to click **Add** and then click on **Jenkins**.
11. Then you will be prompted to the **Jenkins Credentials Provider** page. Under Add Credentials, you can add your **GitHub Username**, **Password**, and **Description**. Then click on **Add**.
12. After returning to the Source Code Management Page, click on Credentials and Choose your **GitHub Credentials**.
13. Keep all the other values as default and select the "**Build**" Tab and inside Goals and options write "**clean package**" and **save** the configuration.

#### Note: The 'clean package' command clears the target directory, Builds the project, and packages the resulting WAR file into the target directory.

16. Now, you will get back to the Maven project "**hello-world**" and click on "**Build Now**" to build the **.war** file for your application

* You can go to **Workspace** > **dist** folder to see that the **.war** file is created there.
* war file will be created in **/var/lib/jenkins/workspace/hello-world/target/**

---------------------------------------------------------------------
### Task-2: Installing and Configuring Tomcat for Deploying our Application on Jenkins Server

* Now, SSH into the Jenkins server (Make sure that you are the root user and Install the Tomcat web server)
* **Note:** (If you are already in Jenkins Server, again SSH is not needed.)

1. Follow below steps:
```
sudo apt update
```
```
sudo apt install tomcat9 tomcat9-admin -y
```
```
ss -ltn
```
when you run `ss -ltn` command you'll see a list of `TCP sockets` that are in a listening state, and the output will include information such as the `local address,` `port,` and the `state of each socket.`
```
sudo systemctl enable tomcat9
```
Now we need to navigate to **server.xml** to change the Tomcat port number from **8080 to 9999**.
(As port number 8080 is already being used by the Jenkins website)
```
sudo vi /etc/tomcat9/server.xml
```
**(Optional step):** If you are unable to open the file then change the permissions by using the below command.
```
sudo chmod 766 /etc/tomcat9/server.xml
```
#### Change 8080 to 9999
* press esc & Enter **":"** and copy paste below code and hit enter
```
g/8080/s//9999/g
```
Save the file using `ESCAPE+:wq!`

* To Verify whether the Port is changed, execute the below Command.
```
cat /etc/tomcat9/server.xml
```
**(Optional step):** If you are unable to open the file then change the permissions by using the below command.
```
sudo chmod 766 /etc/tomcat9/server.xml
```
```
sudo vi /etc/default/tomcat9
```
Paste the path of jdk inside the file
```
JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```
Now restart the system for the changes to take effect
```
sudo service tomcat9 restart
```
```
sudo service tomcat9 status
```
**To exit**, press **ctrl+c**

If any error is present asking for the value of JAVA_HOME
```
vi /etc/default/tomcat9
```
add the following line to the end of the file
```
JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"
```
Now restart the system for the changes to take effect
```
sudo service tomcat9 restart
```
```
sudo service tomcat9 status
```
* Once the Tomcat service restart is successful, go to your web browser and enter **Jenkins Server's Public IP address** followed by **9999** port.

(Example: **http://< Jenkins Public IP >:9999**     or     **http://184.72.112.155:9999**)

* Now you can check the Tomcat running on **port 9999** on the same machine.
* We need to copy the **.war** file created in the previous Jenkins build from the Jenkins workspace to tomcat webapps directory to serve the web content
```
sudo cp -R /var/lib/jenkins/workspace/hello-world/target/hello-world-war-1.0.0.war /var/lib/tomcat9/webapps
```
The above command is copying a `WAR (Web Application Archive)` file from the Jenkins workspace to the Tomcat web apps directory. Let's break down the command:

- `sudo`: Run the command with superuser (root) privileges, as copying files to system directories often requires elevated permissions.

- `cp`: The copy command in Linux.

- `-R`: Recursive option, used when copying directories. It ensures that the entire directory structure is copied.

- `/var/lib/jenkins/workspace/hello-world/target/hello-world-war-1.0.0.war`: Source path, specifying the location of the WAR file in the Jenkins workspace.

- `/var/lib/tomcat9/webapps`: Destination path, indicating the Tomcat webapps directory where the WAR file is being copied.

This command assumes that your Jenkins job has built a WAR file named `hello-world-war-1.0.0.war` in the specified workspace directory. It then copies this WAR file into the Tomcat webapps directory, allowing Tomcat to deploy and run the web application.

* Once this is done, go to your browser and enter Jenkins Public IP address followed by port 9999 and path (URL:  **http://< Jenkins Public IP >:9999/hello-world-war-1.0.0/**).
* Now, you can see that Tomcat is now serving your web page
* Now, Stop tomcat9 and remove it. Otherwise, it will slow down the Jenkins server.
```
sudo service tomcat9 stop
```
```
sudo apt remove tomcat9 -y
```
---------------------------------------------------------------------
**Summary:**
1. Opening the `Jenkins Web Page` on the browser.
2. Unlock Jenkins and create an admin user.
3. Install plugins, including Maven integration.
4. Configure Jenkins to use a specific version of Maven.
5. Create a Maven project in Jenkins for the "hello-world" application.
6. Configure source code management with Git and GitHub.
7. Define build goals and options.
8. Build the project and verify the outcome.
9. Install and configure Apache Tomcat for serving web applications.
10. Deploy the built WAR file to Tomcat.

#### =============================END of LAB-03=============================
