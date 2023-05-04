# How to install and set up Jenkins on EC2 instance

This guide will provide an information on how to install Jenkins on the EC2 instance and create a Master node

## Step 1. Create a new EC2 Instance

1. Go to AWS and launch a new instance
2. You can name it something like `jenkins-master` as it will be your master node
3. For AMI use `Ubuntu 18.04`
4. Use `t2.Medium`
5. For SG you need the following ports:
    * SSH port 22 - for your IP
    * Custom TCP port 8080 - for Jenkins access from anywhere
    * Custom TCP port 43 - access from anywhere to allow connectiong with github
6. Use `tech221.pem` as SSH key
7. Launch the instance

## Step 2. Connect to the instance to install Jenkins

1. Use SSH connection to log in to your instance thourgh GitBash terminal
2. Use the following commands to install all the dependancies:
```
sudo apt-get update
sudo apt-get install default-jdk -y


wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

sudo ufw allow OpenSSH
sudo ufw enable

sudo su
ssh -T git@github.com
```

## Step 3. Create Jenkins account and finish the set-up

1. Connect to Jenkins server in your browser by using `<EC2_publicIP>:8080`
2. It will ask you to confirm the password. In your terminal write `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` in order to retrieve the password, copy and then paste it to Jenkins.
3. Follow the steps to set-up Jenkins, enter your credentials and install recommended packages.
4. Once logged in, go to `Manage Jenkins`
5. Search for `Manage Plugins`
6. Go to `Available Plugins` and then search and install the following plugins:
    * Amazon EC2 plugin
    * NodeJS plugin
    * SSH Agent plugin
7. Then, go back to `Manage Jenkins`
8. Go to `Global Tool Configuration`
9. Scroll down and search for `NodeJS`
10. Add a new version of NodeJS you want to install. For this task I used version 12.1.0
11. Click `Save`

## Step 4. Create jobs on Master Node in order to establish and test CI/CD pipeline

First, we need to establish a `webhook trigger` in order for GitHub to communicate with Jenkins when new code being pushed. For that, we need to go to our GitHub repo -> Settings -> Weebhooks -> Add webhook. There, create a new weebhook with url `http://<jenkinsIP>:8080/github-webhook/` and Content type `application/jason`. 
Also, ensure that you launch your EC2 instance with App installed.

Then, we need to create our first job:
1. Create a new task, for example `spartaApp-ci`, as Freestyle project.
2. You can follow the guide on how to create the job [CI/CD guide](CICDandJenkins.md)
3. In genaral:
    * Set Discard old build to 3
    * GitHub project paste https link to your repo
4. In Source code management:
    * Select Git
    * Paste ssh link to your repo
    * create a credential ssh key
    * Branch to build set to `dev`
5. Build Triggers select `GitHub`
6. Build Environment:
    * Select Provide Node & npm bin/folder to PATH
    * in NodeJS Installation select the node installation we created earlier
7. In Build Steps and shell script:
```
cd app
npm install
npm test
```
8. Click save and test the job if it works

Create a second job to marge code:
1. Create a new task, for example `spartaApp-merge` as Freestyle project
2. You can follow the guide on how to create the job [CI/CD guide](CICDandJenkins.md)
3. In genaral:
    * Set Discard old build to 3
    * GitHub project paste https link to your repo
4. In Source code management:
    * Select Git
    * Paste ssh link to your repo
    * create a credential ssh key
    * Branch to build set to `dev`
    * Add additional behaviour `Merge before build`
    * Name o repository set to origin
    * Branch to merge to set to main
5. Add Post-Build Actions `Git Publisher` and tick boxes for `Push Only if build successful` and `Merge Results`
6. Test the job after you saved it

Create a final job:
1. Create a new task, for example `spartaApp-cd`, as Freestyle project.
2. You can follow the guide on how to create the job [CI/CD guide](CICDandJenkins.md)
3. In genaral:
    * Set Discard old build to 3
    * GitHub project paste https link to your repo
4. In Source code management:
    * Select Git
    * Paste ssh link to your repo
    * create a credential ssh key
    * Branch to build set to `main`
5. Build Environment:
    * Select SSH Agent
    * Create credentils using AWS pem file
7. In Build Steps and shell script:
```
scp -v -r -o StrictHostKeyChecking=no app/ ubuntu@<EC2_publicIP>:/home/ubuntu/
ssh -A -o StrictHostKeyChecking=no ubuntu@<EC2_publicIP> <<EOF
#sudo apt install clear#

cd app
#sudo npm install pm2 -g
# pm2 kill
nohup node app.js > /dev/null 2>&1 &
```
8. Save and test the job


Once all the jobs are working we can link them together to trigger next job when previous one is completed:
1. Go to `spartaApp-ci` configuration and in Post-Build Actions add `Build other projects`, where you want to build `spartaApp-merge`
2. Go to `spartaApp-merge` configuration and in Post-Build Actions add `Build other projects`, where you want to build `spartaApp-cd`
3. Now make some changes in the code and push it to your GitHub. If everything is correct it should trigger all of the jobs one after another and update your app
4. Go to `<EC2_piblicIP>:3000` in order to check if the app is working and updates have been applied