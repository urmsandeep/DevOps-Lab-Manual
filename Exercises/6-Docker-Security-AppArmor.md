# Docker Security with AppArmor and Python

## Objective

The goal of this exercise is to understand how to secure Docker containers using AppArmor profiles with Python for enforcement. 
You will also learn how to apply AppArmor profiles using the Docker SDK for Python and test restricted actions within the container.

## Scenario

You are responsible for deploying a Python-based web application using Docker. 
The application is running inside a Docker container.  You need to secure it to restrict access to sensitive directories 
and prevent unauthorized actions such as executing binaries or accessing restricted files.

The Python Flask application is containerized, and Docker's SDK will be used to apply and verify 
the AppArmor profiles. Finally, you will test restricted actions in the container using Python scripts.

## Pre-requisites

Steps to Install and Use apparmor_parser:
Install AppArmor Utilities: On most Linux distributions, you can install apparmor_parser as part of the apparmor-utils package. To install it, run:

For Ubuntu/Debian-based distributions:

```
sudo apt-get update
sudo apt-get install apparmor-utils
```

## Tasks


### Task 1: Write a Basic Python Flask Application
Create a simple Python Flask application that will be containerized.

File Name: app.py

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, this is a secure Flask application running inside a Docker container!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Task 2: Containerize the Flask Application Using a Dockerfile

Create a *Dockerfile* to containerize the Flask application.

File Name: Dockerfile

```
# Use an official Python runtime as a parent image
FROM python:3.8-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install Flask
RUN pip install flask

# Expose port 5000
EXPOSE 5000

# Run app.py when the container launches
CMD ["python", "app.py"]

```
#### Build a docker image
Image Name: flask-apparmor

```
docker build -t flask-apparmor .
```
#### Output

```
Sending build context to Docker daemon  3.072kB
Step 1/5 : FROM python:3.8-slim
 ---> e83d9d28b2f6
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> 2c4979d5c6f3
Step 3/5 : COPY . /app
 ---> 91d94a789d7a
Step 4/5 : RUN pip install flask
 ---> Running in 7fd028e9370d
Collecting flask
  Downloading Flask-2.0.2-py3-none-any.whl (93 kB)
  ...
Successfully built flask-apparmor
```

### Task 3: Create and Apply AppArmor Profile

Create an AppArmor profile that restricts access to sensitive directories and prevents the execution of certain binaries.

File Name: my-apparmor-profile
AppArmor profiles are stored in the /etc/apparmor.d/ directory. So, you should save the profile as: /etc/apparmor.d/my-apparmor-profile

```
#include <tunables/global>

/usr/bin/python3 {
    # Deny access to sensitive system files
    deny /etc/** r,
    deny /var/** rw,

    # Allow Flask app to bind to port 5000
    network inet stream,

    # Permissions to the application directory
    /app/** rwk,

    # Deny execution of any binaries in /bin or /usr/bin
    deny /bin/** rmix,
    deny /usr/bin/** rmix,

    # Capability restrictions
    capability net_bind_service,
    deny capability sys_admin,
}

```

##### Steps to Apply the Profile:
a. Create the AppArmor profile file: Save the AppArmor profile in /etc/apparmor.d/my-apparmor-profile.
b. Load the Profile: After creating the profile, load it using the following command:

```
sudo apparmor_parser -r /etc/apparmor.d/my-apparmor-profile
```

c. Run the Docker Container with the Profile: When running the Docker container, apply the AppArmor profile with the --security-opt flag:
```
docker run --security-opt="apparmor=my-apparmor-profile" -p 5000:5000 flask-apparmor
```

This filename and location ensure that the AppArmor profile can be correctly loaded and applied to the Docker container.

### Task 4: Use Docker SDK for Python to Apply AppArmor Profile
Write a Python script using the Docker SDK to apply the AppArmor profile to the Flask container.

Python Script (apply_apparmor.py):

```
import docker

# Create a Docker client
client = docker.from_env()

# Build the Docker image
client.images.build(path=".", tag="flask-apparmor")

# Run the container with the AppArmor profile
container = client.containers.run(
    "flask-apparmor",
    ports={'5000/tcp': 5000},
    security_opt=["apparmor=my-apparmor-profile"],
    detach=True
)

print(f"Container started: {container.short_id}")

# Verify AppArmor profile applied
container_info = client.api.inspect_container(container.id)
apparmor_profile = container_info['HostConfig']['SecurityOpt']

print(f"AppArmor profile applied: {apparmor_profile}")

# Stop the container
container.stop()
```

Ensure that you have installed the Docker SDK for Python: If you haven’t installed the Docker SDK yet, run this command:

```
pip install docker
python apply_apparmor.py
```

Expected Output of Running apply_apparmor.py:

```
Building image from Dockerfile...
[INFO] Sending build context to Docker daemon  3.072kB
[INFO] Step 1/5 : FROM python:3.8-slim
 ---> e83d9d28b2f6
[INFO] Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> 2c4979d5c6f3
[INFO] Step 3/5 : COPY . /app
 ---> Using cache
 ---> 91d94a789d7a
[INFO] Step 4/5 : RUN pip install flask
 ---> Using cache
 ---> 4f7e4a98128f
[INFO] Step 5/5 : EXPOSE 5000
 ---> Using cache
 ---> 230dece26d37
[INFO] Successfully built flask-apparmor

Running container with AppArmor profile...

Container started: f8c2a7f9b9b8

Inspecting container to verify AppArmor profile...

AppArmor profile applied: ['apparmor=my-apparmor-profile']

Stopping the container...
```


### Task 5: Test Restricted Actions

Write a Python script to test restricted actions such as accessing /etc/passwd or executing /bin/bash in the container.

Python Script (test_restricted_actions.py):

```
import docker

# Create a Docker client
client = docker.from_env()

# Run the container with the AppArmor profile
container = client.containers.run(
    "flask-apparmor",
    ports={'5000/tcp': 5000},
    security_opt=["apparmor=my-apparmor-profile"],
    detach=True
)

# Test restricted actions
exit_code, output = container.exec_run("cat /etc/passwd")
print(f"Attempt to read /etc/passwd: Exit Code {exit_code}, Output: {output.decode()}")

exit_code, output = container.exec_run("/bin/bash")
print(f"Attempt to execute /bin/bash: Exit Code {exit_code}, Output: {output.decode()}")

# Stop the container
container.stop()
```
Run the script using Python:
```
python test_restricted_actions.py
```

Expected Output of test_restricted_actions.py:

```
Container started: f8c2a7f9b9b8

Attempt to read /etc/passwd: Exit Code 1, Output: 
Attempt to execute /bin/bash: Exit Code 126, Output: 
Container stopped
```

### Questions:

1. What is the purpose of using AppArmor with Docker containers?
2. How do AppArmor profiles help secure a Docker container?
3. Why is it important to restrict access to sensitive directories such as /etc/ and /var/?
4. What other capabilities can you restrict using AppArmor profiles?
5. How can you verify if an AppArmor profile is successfully applied to a Docker container?

### Answers:

1 What is the purpose of using AppArmor with Docker containers? AppArmor is used to enforce security policies and confine applications to a limited set of resources. With Docker containers, it helps to limit access to system resources, files, and networks, thus providing an additional layer of security.

2. How do AppArmor profiles help secure a Docker container? AppArmor profiles define what a containerized application can or cannot do. They restrict access to sensitive directories, network capabilities, file execution, and system calls, ensuring the container behaves securely without affecting the host system.

3. Why is it important to restrict access to sensitive directories such as /etc/ and /var/? Sensitive directories like /etc/ contain configuration files and sensitive information such as user data and system settings. Restricting access prevents the container from reading or modifying important system files, reducing the risk of security breaches.

4. What other capabilities can you restrict using AppArmor profiles? AppArmor can restrict a container's ability to access the network, bind to specific ports, execute binaries, write to specific directories, and use system administration capabilities (cap_sys_admin).

5. How can you verify if an AppArmor profile is successfully applied to a Docker container? You can verify if an AppArmor profile is applied by inspecting the container using the Docker SDK or the Docker CLI. The HostConfig.SecurityOpt field will show the applied security options, including the AppArmor profile name.
