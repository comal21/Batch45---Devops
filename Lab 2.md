## Lab 2: Git and GitHub Operations.

**Objective:**
This lab focuses on `Git` and `GitHub operations,` including `initializing a Git repository,` `making commits,` `creating branches,` and `pushing code to a GitHub repository.`

### Pre-requisites:
1. Create a **GitHub Account** & **Empty Public Repository** with name as **"hello-world"**

   **Note:** (To SignUp for GitHub Account, [Click Here](https://github.com/signup?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home))
2. You need to generate a **Personal Access Token** (PAT) in GitHub.
3. Basic familiarity with **Git Commands.**

Once the GitHub Account and Empty repository is ready, let's operate in the local Anchor Server.

---------------------------------------------------------------------
### Task-1: Initializing the local git repository and committing changes

On the `CICD anchor EC2,` do the below:
```
cd ~/
```
Check the GIT version. 
```
git --version
```
(Optional) If it does not exist, then you can install it with the below command. Else no need to execute the below line:  
```
sudo apt install git -y
```
Download the **Java Code** that we are going to use in the CICD pipeline.
```
 wget https://devopslab-komal.s3.us-east-1.amazonaws.com/hello-world-master+(1).zip
```
```
unzip 'hello-world-master+(1).zip'
```
```
ls
rm hello-world-master.zip
```
```
cd hello-world-master
ls
```
```
git init .
```
To set the `User Identity` ie... `email and user name.` you can use below:

```
git config --global user.email "< Add your email >"
```
```
git config --global user.name "< Add your Name >"
```
```
git status
```
```
git add .
```
```
git status
```
```
git log
```
```
git commit -m "This is the first commit"
```
```
git log
```
```
git status
```

---------------------------------------------------------------------
### Task-2: Pushing the Code to your Remote GitHub Repository  

* To push code to **Github**, You need to generate a **Personal Access Token** (PAT) in github.

**To Generate the Token Follow the below steps:**

* First, Go to your `GitHub homepage,` Click on the top right `profile Icon` and then `settings`
* Click on `Developer settings` (At the bottom on the left side menu). Click on `Personal Access Token` and then Click on `Generate New Token.`
* Under '**Select Scopes**' select all items. Click on '**generate token**'.
* Once generated, **Copy and save the token in a safe location as it is not visible again in GitHub.**

* Create `Alias` as `Origin` to GitHub's Remote repository URL.
```
git remote add origin < Replace your Repository URL > 
```
(**Example:** `git remote add origin https://github.com/janjiralakirankumar/hello-world.git`)

* To view the list of Aliases, run the below command in Terminal.
```
alias
```
* To view a specific alias use the below command.
```
git remote show origin
```
* Now you can push your code to the Remote repository using the below command.
```
git push origin master 
```
* When it asks for a password, enter the **Personal Access Token** and Press Enter (PAT is invisible and It's the expected behavior.)

---------------------------------------------------------------------
### Task 3: Creating a Git Branch and Pushing Code to the Remote Repository
* To create a new Branch
```
git branch dev
```
* To know the current branch
```
git branch
```
* To switch to another branch
```
git checkout dev
```
```
git branch
```
```
vi index.html
```
Press INSERT and add the below content
```
<html>
<body>
<h1> Hi There! This file is added in dev branch </h1>
</body>
</html>
```
Save the file using `ESCAPE+:wq!`
```
git status
```
```
git add index.html
```
```
git status
```
```
git commit -m "adding new file index.html in new branch dev"
```
```
git log
```
```
git push origin dev
```
#### When it asks for a `User ID` enter `GitHub's User ID,` when it asks for a `password` Enter `PAT` and Press Enter. (Note: PAT is invisible when you paste)
```
git branch
```
```
git branch prod
```
```
git branch
```
```
git checkout prod
```
```
git branch
```
```
git merge dev
```
```
git push origin prod
```
#### When it asks for a `User ID` enter `GitHub's User ID,` when it asks for a `password` Enter `PAT` and Press Enter. (Note: PAT is invisible when you paste)
```
git checkout master
```
```
git merge prod
```
```
git push origin master
```
#### When it asks for a `User ID` enter `GitHub's User ID,` when it asks for a `password` Enter `PAT` and Press Enter. (Note: PAT is invisible when you paste)

---------------------------------------------------------------------
**Summary:**
1. Initialize a local Git repository on Anchor EC2 instance.
2. Configure Git settings like email and username.
3. Add and commit changes to the Git repository.
4. Create and switch between Git branches.
5. Make changes and commits in different branches.
6. Merge branches and Push the code to a GitHub repository.

#### =============================END of LAB-02=============================
---
