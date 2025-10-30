# Docker Networking Exercise

## Objective

Understand Docker networking concepts and configure a multi-container application.


## Scenario

Develop a web application with:


* A Python Flask web server (container 1)
* A MySQL database (container 2)
* A Redis cache (container 3)


## Tasks


### Task 1: Create a Bridge Network

**```docker network create --driver bridge my-bridge-net```**

Output:
87c23b491f5994c74e497f4f4f4f4f4f4f4f4

### Task 2: Verify the Network*

**```docker network ls```**

Output:

NETWORK ID     NAME           DRIVER    SCOPE
87c23b491f59   my-bridge-net   bridge    local

### Task 3: Inspect the Network*

**```docker network inspect my-bridge-net```**

Output:
[
    {
        "Name": "my-bridge-net",
        "Id": "87c23b491f5994c74e497f4f4f4f4f4f4f4f4",
        "Created": "2023-02-20T14:30:45.421654Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]

### Task 4: Launch Containers

Create a `Dockerfile` for the Flask app:

## Python code. Save this as app.py

```
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/about', methods=['GET'])
def about():
    return jsonify({
        "name": "Simple REST API",
        "version": "1.0",
        "description": "This is a simple REST API built with Flask."
    })

if __name__ == '__main__':
    app.run(debug=True, port=5001)  # Specify the port number here
```

## Requirements: Save this as requirements.txt
```
Flask==2.0.1
```

## Dockerfile

```
# Use the official Python image from the Docker Hub
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file and app code to the container
COPY requirements.txt .
COPY app.py .

# Install Flask (and any other dependencies you might have)
RUN pip install --no-cache-dir -r requirements.txt

# Expose the port the app runs on
EXPOSE 5001

# Define the command to run the application
CMD ["python", "app.py"]

```

### Build the Flask image:

**```docker build -t flask-api .```**

Output:

Sending build context to Docker daemon  3.584kB
Step 1/5 : FROM python:3.9-slim

## Launch the containers: (-d denotes dettach mode i.e. container runs in background)

```
    docker run -d --name mysql --net=my-bridge-net mysql:latest
    docker run -d --name redis --net=my-bridge-net redis:latest
    docker run -d --name flask --net=my-bridge-net -p 5001:5001 flask-api
```

Output:

mysql container ID: 237c941f4f4f
redis container ID: 456c941f4f4f
flask container ID: 678c941f4f4f


### Task 5: Test Connectivity

Exec into the Flask container:

```
docker exec -it flask bash
```

Ping the MySQL container:

```
ping mysql
```

Output:

PING mysql (172.18.0.2) 56(84) bytes of data.
64 bytes from mysql (172.18.0.2): icmp_seq=1 ttl=64 time=0.078 ms


# Ping the Redis container:

```
ping redis
```

Output:

PING redis (172.18.0.3) 56(84) bytes of data.
64 bytes from redis (172.18.0.3): icmp_seq=1 ttl=64 time=0.078 ms

### Task 6: Clean up

Stop and remove containers:

```
docker stop mysql redis flask && docker rm mysql redis flask
```

# Remove the network

```
docker network rm my-bridge-net
```

Output:

my-bridge-net

### Questions:

1. What is the purpose of the --net flag in docker run?
2. How do containers communicate with each other on the same network?
3. What is the difference between a bridge network and a host network?
4. How can you expose a container's port to the host machine?


### Answers:

1. The --net flag specifies the network to connect the container to.
2. Containers communicate using their container names or IP addresses.
3. A bridge network provides isolation between containers, while a host network shares the host's network stack. Think of it like:
- Bridge Network: Containers are in a private room, talking to each other through a door.
- Host Network: Containers are in the same room as the host, talking directly to everyone.
4. Use the -p flag to expose a container's port (e.g., `-p 500")
