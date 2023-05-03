# CI/CD and Jenkins

### Testing CI via dev branch and merging to main. Test_02

## What is CI

CI - automates the process of merging/commiting the code to master branch. It automatically builds and tests the new code as it's being merged. Helpes to eliminate any errors or security problems on the early stages.

## Difference between CD and CDE

CD - works together with CI. After code was tested amd merged, CD packages the software and ensures that it is ready to be deployed into production for a client to use.

CDE - allows the deployment of the app automatically, without the need for human intervention. DevOps sets the criteria for code releases, and when those criterias are met and validated, the code is deployed into the production environment.

Main difference is that in CD the deployment is done manually.


## Jenkins and other tools

![CI/CD tools](resources/CICD_tools.webp)

Tools for CI/CD pipiline:

* Jenkins
* circleci
* TeamCity
* Bamboo
* GitLab

We will be using Jenkins!

Jenkins is an open source automation server in which the central build and CI process take place.

![Jenkins diagram](resources/Jenkins_diagram.webp)


Benefits of Jenkins:
* range of plugins
* easy installation and user-friendly
* huge community and resources
* easy environment configuration


## Connect Jenkins to GitHub repo and post a job to test the app

![CICD diagram](resources/CICD.png)

### Step 1. Create new SSH key

1. Open `GitBash` terminal
2. `cd .ssh` - navigate into .ssh folder
3. Create a new key by using `sss-keygen -t rsa -b 4096 -C "olegfursa93@hotmail.com"
4. Type the name, for example `oleg_jenkins_key`, and then press `Enter` twice to skip the password set up
5. Now if you use `ls` you should be able to see new keys created

### Step 2. Add public key to GitHub repo
1. Create a new repo on GitHub, or open an excisting repo with `Sparta App`
2. Go in to project settings:

![Project Settings](resources/project_settings.JPG)

3. On the left panel search for `Deploy keys`
4. Click on `Add deploy key`
5. Name your new key
6. In bash terminal, inside the `.ssh` folder, use `cat oleg_jenkins_key.pub` in order to display the content of the public key
7. Copy the public key and paste it inside your `Deploy key` on GitHub
8. Save the key


### Step 3. Create a job on Jenkins

1. Log in to Jenkins account
2. Click on `New Item` to create a new job:

![New job jenkins](resources/jenkins_new_item.JPG)

3. Give a name to the job, select `Freestyle project` and click `OK`:

![New task name jenkins](resources/jenkins_name_new_job.JPG)

4. Set up general settings:

    ![General settings jenkins](resources/jenkins_general_settings.JPG)

    * `Description` - write a short description of the job
    * `Discard old builds` - set `Max # of builds to keep` to `3`, meaning it will only keep 3 copies of the same job once built, and delete the older once
    * `GitHub project` - copy an `https` link from your GitHub project

5. Set up Office 365 Connector:

    ![Office 365 connector jenkins](resources/jenkins_office365_connector.JPG)

    * Select `Restrict where this project can be run` and type `sparta-ubuntu-node`, meaning it will create a new node outside of master node in order to run the job

6. Set up Source code management:

    ![Source code management jenkins](resources/jenkins_source-code_manager.JPG)

    Here we establish connection with GitHub, as thats where our code will be managed:

    * Select `Git`
    * In `Repository URL` copy an `ssh` link from your GitHub project
    * In `Credentials` select the key. If you don't have a key created yet, click on `Add` and then `Jenkins`:

        ![Credential provider jenkins](resources/jenkins_credential_provider.JPG)

            1. In `Kind` select `SSH Username with private key`
            2. In `username` enter your key name
            3. In `Private key` select `Enter directly`
            4. Click on `Add`
            5. In bash terminal, use `cat oleg_jenkins_key` in order to display the content of your private key
            6. Copy all of the content from the key
            7. Paste the content into Jenkins
            8. Click `Add`
    * In `Branches to build` change it to `main`

7. Set up Build Environment:

    ![Build Environment jenkins](resources/jenkins_build_environment.JPG)

    * Select `Provide Node & npm bin/folder to Path`
    * In `NodeJS Installation` select `Sparta-Node-JS` 

8. Set up Build:

    ![Build jenkins](resources/jenkins_build.JPG)

    * Click on `Add build step`
    * Select `shell script`
    * Add following commands:
    ```
    cd app
    npm install
    npm test
    ```

9. Post-build Actions (Optional):
    * Here you can add an action to be executed after this job is finished, for example start another job if this one was successful

10. Click on `Save`

### Step 4. Run the project

1. Once job is created, click on `Build Now` in order to run the job:

    ![Build project jenkins](resources/jenkins_job_created.JPG)

2. In `Build History` you can see the status of the project:

    ![Build History jenkins](resources/jenkins_build_history.JPG)

    * It will show you the time when it was built
    * There is a sphere that shows the status of the job:
        * red - there is an error
        * grey - pending/executing
        * blue - completed successfully

3. If you click on the sphere you can see the console output:

    ![Console output jenkins](resources/jenkins_console_output.JPG)




## Creating a webhook trigger

1. Open your GitHub project
2. Go into project `Settings`
3. In the settings search for `Webhooks`
4. Click `Add webhook`
5. In the webhook settings:
    * Payload URL - copy Jenkins url
    * Content type - chose `application/json`
    * Select `Just the push event`
    * Click on `Add webhook`
6. Go to Jenkins
7. Open your task configuration
8. In Build triggers select `GitHub hook trigger for GITScm polling`:

    ![Build trigger jenkins](resources/jenkins_build_trigger.JPG)
9. Now, if you push the changes to your GitHub it should automatically trigger the test on Jenkins


## Automating CI pipeline

### Step 1. Create a new branch on GitHub

1. Open your Project in VS code
2. In the Git terminal type `git branch dev` to create a dev branch
3. Use `git checkout dev` to switch to `dev` branch by default
4. Now, use `git push -u origin dev` to push your changes to dev branch

### Step 2. Create a job to test a code from dev branch

1. Go to your Jenkins
2. Create a new task. As it will be identical to the previous task we've created we could use that as a template to copy settings from it
3. In `Source Code Management` change `Branches to build` to `dev`:

    ![Dev branch source](resources/jenkins_source_dev.JPG)

4. The rest you can leave as it is. Click `Save`
5. Use `Build Now` in order to test the job manually

### Step 3. Merge dev branch with main branch

1. Create a new task on Jenkins
2. In `General`:

    ![Merge task general settings](resources/jenkins_merge_general.JPG)

    * Write a description
    * Set `Discard old builds` to 3
    * `GitHub Project` copy https link to your project and paste it there

3. In `Source Code management` copy the settings from the previous task. Then click on `Add` in Additional Behaviours and select `Merge before build`:

    ![Merge before build settings](resources/jenkins_merge_source_management.JPG)

    * `Name of repository` - set to `origin`
    * `Branch to merge to` - set to `main`
    * Rest leave to default

4. In `Post-build Action` click on `Add` and select `Git Publisher`:

    ![Git Publisher settings](resources/jenkins_merge_postbuild.JPG)

    * Enable `Push only if build successful`
    * Enable `Merge Results`

5. Click on save
6. Use `Build Now` in order to test the job manually

### Step 4. Trigger merge job if test is successful

1. Go back to your `CI` job configuration
2. Scroll down to `Post-build actions` and add `Build other projects`
3. In `projects to build` type the name of the project you want to run, for example `oleg_ci_merge`

    ![Post-build action to trigger merge](resources/jenkins_postbuild_action.JPG)


4. Save your project
5. Now, to test if it works, push some changes form your local repo and it should trigger the test first, and if the test is successful it will then merge the changes from dev branch to main branch and push the changes to your GitHub

