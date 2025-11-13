## Creating "Hello World" Jenkins Job

### Step 1: Create a file (hello-world.sh) in your GitHub account.
Here’s how you can create the hello-world.sh script and add it to a GitHub repository.

#### (1a)  Create a New GitHub Repository
1. Log in to your **GitHub account.** (www.github.com)
2. Click the + icon (top-right) → New Repository.
3. Fill in the repository details:
4. Repository Name: **devops-sample-code**
5. Description: A demo repository for Jenkins scripting.
6. Visibility: **Public** keep it public
7. Click Create Repository.

#### (1b) Create a fine grained personal access token to use for commiting code to your github
Refer to instruction here:
https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token


#### (2) Create the Script Locally
1. Open a terminal on your laptop or system
2. Create a new file named hello-world.sh:
```
touch hello-world.sh
```
3. Edit the file (using vim, vss or notepad) and add the following content:
```
#!/bin/bash
echo "Hello, Jenkins!"
```
4. Make the script executable
```
chmod +x hello-world.sh
```
#### (3)  Initialize a Local Git Repository
If this is your first time using Git locally, configure it and initialize the new repository.
```
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git init
```
#### (4) Add the Script to the Repository

1. Add the script to the repository:
```
git add hello-world.sh
```
2. Verify git status

```
git status
```
You will see the new file ready to be commited
```
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   hello-world.sh
```

2. Commit the changes:
```
git commit -m "Add hello-world.sh"
```

The file will added as changed.

```
[master (root-commit) 2e1b718] Add hello-world.sh
 1 file changed, 2 insertions(+)
 create mode 100644 hello-world.sh
```

### (5) Link the Local Repository to GitHub
1. Copy the remote URL of the GitHub repository you created. It will look like this:
```
https://github.com/<your-GitHub-account>/devops-sample-code.git
```

2. Link the local repository to the GitHub repository:
```
git remote add origin https://github.com/<your-GitHub-username>/devops-sample-code.git
```

### (6) Push the Script to GitHub
1. Push the changes to the remote repository:
```
git push -u origin main
```
It will prompt for your GitHub username and Personal Access Token (PAT) that you had created in Step 1(b)

Username for 'https://github.com': <your-GitHub-account>
Password for 'https://<your-GitHub-account>@github.com': <Copy/paste the PAT token than you had created>
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 306 bytes | 30.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/urmsandeep/devops-sample-code.git
   4be1c56..1b647f7  main -> main
branch 'main' set up to track 'origin/main'.

### (7) Verify the Script on GitHub
1. Go to your GitHub repository URL: https://github.com/<your-GitHub-username>/DevOps.
2. Navigate to the hello-world.sh file under your repository structure.

## Pre-requisite: Ensure Jenkins is running
Refer to this page:

https://github.com/urmsandeep/DevOps/blob/main/Exercises/Jenkins-CI-Automation.md#getting-started-with-jenkins

Once you have Jenkins running, follow these steps to create and configure first Jenkins job in Jenkins.

---

## Step 1: Access Jenkins
1. Open your browser and navigate to the Jenkins URL (e.g., `http://localhost:8080`).
2. Log in using your credentials.

---

## Step 2: Create a New Job
1. On the Jenkins dashboard, click **"New Item"** in the left-hand menu.
2. Enter a name for your job (e.g., `HelloWorld`).
3. Select **Freestyle project**.
4. Click **OK** to proceed.

---

## Step 3: Configure the Job

### General Settings
1. Add a description for the job:
   ```text
   Hello World! Jenkins job.

### Source Code Management
Select Git under Source Code Management.
Enter the repository URL: https://github.com/<your-username>/repo.git

### Build Steps
Under Build, click Add build step → Execute shell.
Add the following shell script
```
sh hello-world.sh
```
### Step 4: Save and Run the Job
Click Save to save your job configuration.
On the job page, click Build Now to run the job.

### Build Triggers
To trigger build click on Jenkins Dashboard -> HelloWorld -> Build Now
This will trigger the build job.

#### Step 5: View the Build Output
Navigate to the Build History section on the left-hand side of the job page.
Click the build number (e.g., #1).
Click Console Output to see the build logs.

```
Started by user Admin
Building in workspace /var/jenkins_home/workspace/HelloWorld
[HelloWorld] $ /bin/sh -xe /tmp/jenkins1281930102.sh
+ echo 'Hello, Jenkins!'
Hello, Jenkins!
Finished: SUCCESS
```
