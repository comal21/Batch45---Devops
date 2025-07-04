Connect to the jenkins server.
```
cd /var/lib/jenkins/
```
```
sudo nano jenkins.model.JenkinsLocationConfiguration.xml
```
In this file change the URL of the Jenkins server
and restart Jenkins using:
```
sudo service jenkins restart
```
Steps to Upgrade from t2.micro to t3.medium
1. Stop the EC2 Instance
Go to AWS Console → EC2 → Instances
Select your t2.micro instance
Click Instance state → Stop instance

2. Change Instance Type
Click Actions → Instance settings → Change instance type
Select t3.medium
Click Apply

3. Start the Instance
Click Instance state → Start instance
