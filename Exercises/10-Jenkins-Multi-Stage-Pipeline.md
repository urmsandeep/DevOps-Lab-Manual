# Jenkins Multi-Stage Pipeline Exercise: Deploying a Python Application

## Objective
In this exercise, we will create and execute a **Jenkins Pipeline** that performs multiple stages: **Build**, **Test**, and **Deploy** 
for a sample Python application. This will help us understand how to implement and manage multi-stage CI/CD pipelines in Jenkins.

---

## Scenario
You are tasked with automating the CI/CD pipeline for a Python Flask application. The pipeline should perform the following stages:
1. **Build**: Install dependencies.
2. **Test**: Run unit tests to ensure the application works as expected.
3. **Deploy**: Deploy the application by copying it to a mock deployment directory.

---

## Steps to Complete the Exercise

### Step 1: Set Up Jenkins
1. Install Jenkins and ensure it is running on your local machine or a server.
2. Install the following plugins:
   - **Pipeline Plugin** (if not already installed).

---

### Step 2: Create a Sample Python Application
1. Create a directory for the application:
   ```
   mkdir python-flask-app
   cd python-flask-app

2. Create the following files:
file: app.py
```
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Jenkins Multi-Stage Pipeline!"

if __name__ == "__main__":
    app.run(debug=True)
```
file: requirements.txt:
```
flask==2.1.2
```
file: test_app.py
```
import unittest
from app import app

class TestApp(unittest.TestCase):
    def test_home(self):
        tester = app.test_client()
        response = tester.get("/")
        print(response.data.decode("utf-8"))
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data.decode("utf-8"), "Hello, Jenkins Multi-Stage Pipeline!")

if __name__ == "__main__":
    unittest.main()
```
### Step 3: Push the Application to a Git Repository
Push the repository to your GitHub.
```
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/<your-GitHub-username>/devops-sample-code.git
git push -u origin main
```

### Step 4: Create a Multistage Jenkins Pipeline
Go to your Jenkins dashboard and click "New Item".
Enter a name (e.g., Python-MultiStage-Pipeline) and select Pipeline.
Click OK.

### Step 5: Define the Jenkinsfile
In the project directory, create a file named Jenkinsfile with the following content:

```
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'python3 -m unittest discover -s .'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                mkdir -p ${WORKSPACE}/python-app-deploy
                cp ${WORKSPACE}/app.py ${WORKSPACE}/python-app-deploy/
                '''
            }
        }
        stage('Run Application') {
            steps {
                echo 'Running application...'
                sh '''
                nohup python3 ${WORKSPACE}/python-app-deploy/app.py > ${WORKSPACE}/python-app-deploy/app.log 2>&1 &
                echo $! > ${WORKSPACE}/python-app-deploy/app.pid
                '''
            }
        }
        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh '''
                python3 ${WORKSPACE}/test_app.py
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}
```
Push the Jenkinsfile to your Git repository:

```
git add Jenkinsfile
git commit -m "Added Jenkinsfile"
git push
```

### Step 6: Configure the Jenkins Job
In Jenkins, go to the pipeline configuration:
Under Pipeline, select Pipeline script from SCM.
Choose Git as the SCM and enter your repository URL.
Add credentials if necessary.
Save the configuration.

### Step 7: Run the Pipeline
Click Build Now to start the pipeline.
Observe the stages (Build, Test, Deploy) as they execute.
Check the console output for logs and results.

### Step 8: Handling build errors
When you run the Jenkins job, you will run into build errors in the pipeline, specifically around
missing python3, pip and other dependencies. To resolve this, directly log into the Jenkins docker
using the command:

```
docker ps -a
CONTAINER ID   IMAGE                    COMMAND                  CREATED        STATUS                    PORTS
                                                                                                                NAMES
78a4587c55da   jenkins/jenkins:lts      "/usr/bin/tini -- /u…"   8 hours ago    Up 8 hours                0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp             
```
Then connect into the docker using:
```
docker exec -it -u root <container-id> bash
```
This will take into the shell of the docker instance and run commands listed below inside the container shell

```
root@78a4587c55da:/#

apt-get update
apt install python3
apt install pip
apt install python3.11-venv
apt install python3-flask
python3 -m unittest discover -s .
```

### Expected Outcome
Build Stage:
- The pip install -r requirements.txt command installs Flask and other dependencies.
Test Stage:
- The unittest framework runs the test cases and ensures they pass.
Deploy Stage:
- The application file (app.py) is copied to /tmp/python-app-deploy.

Multi-stage successfully runs as follows:
```
Started by user admin
Obtained Jenkinsfile from git https://github.com/urmsandeep/devops-sample-code.git
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/Python-MultiStage-Pipeline
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/Python-MultiStage-Pipeline/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/urmsandeep/devops-sample-code.git # timeout=10
Fetching upstream changes from https://github.com/urmsandeep/devops-sample-code.git
 > git --version # timeout=10
 > git --version # 'git version 2.39.5'
 > git fetch --tags --force --progress -- https://github.com/urmsandeep/devops-sample-code.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision f82e59e16ad6e72100e54d776df07dd26edf80de (refs/remotes/origin/main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f f82e59e16ad6e72100e54d776df07dd26edf80de # timeout=10
Commit message: "Updated Jenkinsfile"
 > git rev-list --no-walk 7733cb39ce899622faf875a95ee6a5b39effa66c # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] echo
Creating virtual environment and installing dependencies...
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] echo
Running tests...
[Pipeline] sh
+ python3 -m unittest discover -s .
.
----------------------------------------------------------------------
Ran 1 test in 0.004s

OK
Hello, Jenkins Multi-Stage Pipeline!
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy)
[Pipeline] echo
Deploying application...
[Pipeline] sh
+ mkdir -p /var/jenkins_home/workspace/Python-MultiStage-Pipeline/python-app-deploy
+ cp /var/jenkins_home/workspace/Python-MultiStage-Pipeline/app.py /var/jenkins_home/workspace/Python-MultiStage-Pipeline/python-app-deploy/
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Run Application)
[Pipeline] echo
Running application...
[Pipeline] sh
+ echo 5064
+ nohup python3 /var/jenkins_home/workspace/Python-MultiStage-Pipeline/python-app-deploy/app.py
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test Application)
[Pipeline] echo
Testing application...
[Pipeline] sh
+ python3 /var/jenkins_home/workspace/Python-MultiStage-Pipeline/test_app.py
.
----------------------------------------------------------------------
Ran 1 test in 0.004s

OK
Hello, Jenkins Multi-Stage Pipeline!
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
Pipeline completed successfully!
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

### Additional Enhancements
Add a Code Quality Analysis stage using tools like flake8 or pylint.
```
stage('Code Quality') {
    steps {
        echo 'Running code quality checks...'
        sh 'flake8 .'
    }
}
```
Deploy the application to a Docker container as part of the pipeline.
Integrate notifications (e.g., email or Slack) for pipeline success or failure.


