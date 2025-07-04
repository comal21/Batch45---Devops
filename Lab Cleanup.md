## Clean Up

Login to the JumpServer and navigate to the below directory
```
cd ~ && cd devops-labs
```
Execute the below command to terminate you Docker Server, Jenkins Server and delete the Key that was crated on AWS
```
terraform destroy -auto-approve
```

Now exit from the JumpServer
```
exit
```
Now navigate to your AWS Managment Console and manually terminate your JumpServer.
