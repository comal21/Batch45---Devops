## Setup the `Docker Server` and `Jenkins Server` for CICD Lab.

### Task 1: Manually launch the Jump Server EC2 Instance

* Region: North Virginia (us-east-1).
* Use tag Name: `Jump-Server`
* AMI Type and OS Version: `Ubuntu 24.04 LTS`
* Instance type: `t2.micro`
* Create key pair with name: `DevOps-Keypair`
* Create security group with name: `DevOps-SG`
   (Include Ports: `22 [SSH],` `80 [HTTP],` `8080 [Jenkins],` `9999 [Tomcat],` and `4243 [Docker]`)
* Configure Storage: 10 GiB
* Click on `Launch Instance.`


### Task 2: Installing Terraform onto `Jump Server` to automate the creation of 2 more EC2 instances.
Once the Jump Server is up and running, SSH into the machine using `MobaXterm` or `Putty` with the username `ubuntu` and do the following:

```
sudo hostnamectl set-hostname JumpServer
bash
```
```
sudo apt update
```
```
sudo apt install wget unzip -y
```
```
wget https://releases.hashicorp.com/terraform/1.9.2/terraform_1.9.2_linux_amd64.zip
```

```
unzip terraform_1.9.2_linux_amd64.zip
ls
```
```
sudo mv terraform /usr/local/bin
```
```
rm terraform_1.9.2_linux_amd64.zip
```
```
terraform -v
```

### Task 3: Install Python 3, pip, AWS CLI, and Ansible on to Jump Server
Install Python 3 and the required packages:
```
sudo apt install python3-pip -y
```
Installing `AWS CLI`
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
Installing Ansible
```
sudo apt install ansible -y
```
For Authentication with AWS we need to provide `IAM User's CLI Credentials`
```
aws configure
```
To Generate the credentials:

![image](https://github.com/user-attachments/assets/ec0592f6-962e-4192-8279-e0412a94fd7b)
![image](https://github.com/user-attachments/assets/04639e13-50cc-4969-9a77-c054e369a2f5)
![image](https://github.com/user-attachments/assets/a9d705c3-9b8e-44c7-a1e9-7aacfbdcd3b7)

Once configured, do a smoke test to check if your credentials are valid and got the access to AWS account.
```
aws s3 ls
```
(Or)
```
aws iam list-users
```

### Task 4: Create a host inventory file with the necessary permissions (Ansible need this Inventory file for identifying the targets)
```
sudo mkdir /etc/ansible && sudo touch /etc/ansible/hosts
```
Change the permissions. The below command gives `Read, Write and Execute` permissions to `Owner,` `Read and Write` permissions to the `Group,` and `Read and Write` permissions to `Others.`
```
sudo chmod 766 /etc/ansible/hosts
```
Cross verify using the below command
```
ls -la /etc/ansible/
```


### Task 5: Utilizing Terraform, initiate the deployment of two new servers: `Docker-server` and `Jenkins-server` 

As a first step, create a keyPair using `ssh-keygen` Command.

**Note:**
1. This will create `id_rsa` and `id_rsa.pub` in Jump Machine in `/home/ubuntu/.ssh/` path.
2. While creating choose defaults like:
   * path as **/home/ubuntu/.ssh/id_rsa**
   * don't set up any passphrase, and just hit the '**Enter**' key for 3 questions it asks.
3. `-t rsa:` Specifies the type of key to create, in this case, RSA.
4. `-b 2048:` Specifies the number of bits in the key, 2048 bits in this case. The larger the number of bits, the stronger the key.
```
ssh-keygen -t rsa -b 2048 
```

Now Create the terraform directory and set up the config files in it
```
mkdir devops-labs && cd devops-labs
```
```
vi main.tf
```
Copy and paste the below code into `main.tf`
```
provider "aws" {
  region = var.region
}

resource "aws_key_pair" "mykeypair" {
  key_name   = var.key_name
  public_key = file(var.public_key)
}

# to create 2 EC2 instances
resource "aws_instance" "my-machine" {
  # Launch 2 servers
  for_each = toset(var.my-servers)

  ami                    = var.ami_id
  key_name               = var.key_name
  vpc_security_group_ids = [var.sg_id]
  instance_type          = var.ins_type
  depends_on             = [aws_key_pair.mykeypair]

  # Read from the list my-servers to name each server
  tags = {
    Name = each.key
  }

  # Ensure the .ssh directory exists before copying the file
  provisioner "remote-exec" {
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("/home/ubuntu/.ssh/id_rsa")  # Replace with the path to your private key
      host        = self.public_ip
    }

    inline = [
      "mkdir -p /home/ubuntu/.ssh"   # Ensure .ssh directory exists before copying the file
    ]
  }

  provisioner "file" {
    source      = "/home/ubuntu/.ssh/id_rsa"   # Source path on Host machine
    destination = "/home/ubuntu/.ssh/id_rsa"  # Destination path on EC2 instance
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("/home/ubuntu/.ssh/id_rsa")  # Replace with the path to your private key
      host        = self.public_ip
    }
  }

  # After the file is copied, change its permissions
  provisioner "remote-exec" {
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("/home/ubuntu/.ssh/id_rsa")  # Replace with the path to your private key
      host        = self.public_ip
    }

    inline = [
      "chmod 400 /home/ubuntu/.ssh/id_rsa"  # Change the permissions of the copied file after it's been transferred
    ]
  }

  provisioner "local-exec" {
    command = <<-EOT
      echo [${each.key}] >> /etc/ansible/hosts
      echo ${self.public_ip} >> /etc/ansible/hosts
    EOT
  }
}
```
Now, create the variables file with all variables to be used in the `main.tf` config file.
```
vi variables.tf
```
```
### **Note:** Change the following Inputs in `variables.tf.`

### Edit the **Allocated Region** (**Ex:** ap-south-1) & **AMI ID** of same region,
### Replace the same **Security Group ID** Created for the Jump Server
### Add your Name for **KeyPair** ("**YourName**-CICDlab-KeyPair")

variable "region" {
    default = "us-east-2"
}

# Change the SG ID. You can use the same SG ID used for your CICD Jump server
# Basically the SG should open ports 22, 80, 8080, 9999, and 4243
variable  "sg_id" {
    default = "sg-02b8781c33d6f584d" # us-east-1
}

# Choose a free tier Ubuntu AMI. You can use below. 
variable "ami_id" {
    default = "ami-0e86e20dae9224db8" # us-east-1; Ubuntu
}

# We are only using t2.micro for this lab
variable "ins_type" {
    default = "t2.micro"
}

# Replace 'yourname' with your first name
variable key_name {
    default = "Mehar-Jenkins-Docker-KeyPair"
}

variable public_key {
    default = "/home/ubuntu/.ssh/id_rsa.pub"   #Ubuntu OS
}

variable "my-servers" {
  type    = list(string)
  default = ["Jenkins-Server", "Docker-Server"]
}
```
Now, execute the terraform commands to launch the new servers
```
terraform init
```
```
terraform plan
```
```
terraform apply -auto-approve
```
Once the Changes are Applies, Go to `EC2 Dashboard` and check that `2 New Instances` are launched. Also check the `inventory file` and ensure the below output.
```
cat /etc/ansible/hosts
```

### Task 6: Check the access from `Jump to Jenkins` and `Jump to Docker`

From `Jump Server` SSH into `Jenkins-Server`, check they are accessible.

```
ssh ubuntu@<Jenkins ip address>
```
Set the hostname
```
sudo hostnamectl set-hostname Jenkins
bash
```
```
sudo apt update
```
Exit from the Jenkins server
```
exit
```
```
exit
```

Now from `Jump Server` SSH into `Docker-Server` and check they are accessible.
```
ssh ubuntu@<Docker ip address>  
```
Set the hostname as
```
sudo hostnamectl set-hostname Docker
bash
```
```
sudo apt update
```
Exit only from the Docker Server, not the Jump Server.
```
exit
```
```
exit
```


### Task 7: Use `Ansible` to deploy respective packages onto each of the servers 
In Jump Server Create a directory and change to it
```
cd ~ && mkdir ansible && cd ansible
```
Now, Create a playbook, which will deploy packages onto the `Docker-server` and `Jenkins-Server.` For this Create a new File with the name `main.yml.`
```
vi main.yaml
```
Copy and paste the below code and save it.
```
---

- name: Start installing Jenkins pre-requisites before installing Jenkins
  hosts: Jenkins-Server
  become: yes
  become_method: sudo
  gather_facts: no

  tasks:

  - name: Update apt repository with latest packages
    apt:
      update_cache: yes
      upgrade: yes

  - name: Installing jdk17 in Jenkins server
    apt:
      name: openjdk-17-jdk
      update_cache: yes
    become: yes

  - name: Installing jenkins apt repository key
    apt_key:
      url: https://pkg.jenkins.io/debian/jenkins.io-2023.key
      state: present
    become: yes

  - name: Configuring the apt repository
    apt_repository:
      repo: deb https://pkg.jenkins.io/debian binary/
      filename: /etc/apt/sources.list.d/jenkins.list
      state: present
    become: yes

  - name: Update apt-get repository with "apt-get update"
    apt:
      update_cache: yes

  - name: Finally, its time to install Jenkins
    apt: name=jenkins update_cache=yes
    become: yes

  - name: Jenkins is installed. Lets start 'Jenkins' now!
    service: name=jenkins state=started


  - name: Wait until the file /var/lib/jenkins/secrets/initialAdminPassword is present before continuing
    wait_for:
      path: /var/lib/jenkins/secrets/initialAdminPassword
  - name: You can find Jenkins admin password under 'debug'
    command: cat /var/lib/jenkins/secrets/initialAdminPassword
    register: out
  - debug: var=out.stdout_lines


- name: Start the Docker installation steps
  hosts: Docker-Server
  become: yes
  become_method: sudo
  gather_facts: no

  tasks:

  - name: Update 'apt' repository with latest versions of packages
    apt:
      update_cache: yes

  - name: install docker prerequisite packages
    apt:
      name: ['ca-certificates', 'curl', 'gnupg', 'lsb-release']
      update_cache: yes
      state: latest

  - name: Install the docker apt repository key
    apt_key: url=https://download.docker.com/linux/ubuntu/gpg state=present
    become: yes

  - name: Configure the apt repository
    apt_repository:
      repo: deb https://download.docker.com/linux/ubuntu bionic stable
      state: present
    become: yes

  - name: Update 'apt' repository
    apt:
      update_cache: yes

  - name: Install Docker packages
    apt:
      name: ['docker-ce', 'docker-ce-cli', 'containerd.io']
      update_cache: yes
    become: yes

  - name: Install jdk17 in Docker server. Maven needs this.
    apt:
      name: openjdk-17-jre-headless
      update_cache: yes
    become: yes

  - name: Start Docker service
    service:
      name: docker
      state: started
      enabled: yes


  - lineinfile:
       dest: /lib/systemd/system/docker.service
       regexp: '^ExecStart='
       line: 'ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock'


  - name: Reload systemd
    command: systemctl daemon-reload

  - name: docker restart
    service:
      name: docker
      state: restarted
...
```
Run the above playbook to deploy the packages onto target servers.
```
ansible-playbook main.yaml
```


### Task 8 : Verify the Jenkins and Docker landing page.
Verify the Jenkins Landing Page: Open a web browser and navigate to your Jenkins landing page using your Jenkins server's public IP address. Replace `<Your_Jenkins_IP>` with the actual public IP.
```
http://<Your_Jenkins_IP>:8080/
```
Verify the Docker Landing Page: Open a web browser and access the Docker landing page using your Docker server's public IP address. Replace `<Your_Docker_IP>` with the actual public IP.
```
http://<Your_Docker_IP>:4243/version
```
---------------------------------------------------------------------
**Summary:**
1. Launch two EC2 instances in AWS - one for Jenkins and one for Docker.
2. Install Terraform on the Jenkins server to automate infrastructure provisioning.
3. Configure AWS CLI and Ansible for managing resources.
4. Create a Terraform configuration to define the servers and their attributes.
5. Launch the servers using Terraform.
6. Update the Ansible hosts file with the server details.
7. Configure Jenkins and Docker servers with proper host names.
8. Use Ansible to install necessary software packages and dependencies on both servers.

#### =============================END of LAB-01=============================


