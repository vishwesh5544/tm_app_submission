# tm_app_submission
Graded Project on Travel Memory Application Deployment Submission

## Overview
The entire solution application is forked to my github profile and the link is as follows: [TravelMemory fork](https://github.com/vishwesh5544/TravelMemory)

I setup a github actions workflow to dockerize the FE and BE and then finally push to my docker hub. The links to public repos are as follows:
1. [Frontend](https://hub.docker.com/repository/docker/vishwesh23/travel-memory-devops7-frontend/general)
2. [Backend](https://hub.docker.com/repository/docker/vishwesh23/travel-memory-devops7/general)

After it is pushed to dockerhub, I manually ssh into the instance and set it up with reverse proxy to point to docker containers.
Once a single instance is setup, I then created an AMI to spin up multiple instances.
Finally, used a target group to point to instances and setup an ELB to point to that target group.

## Task 1 - Backend Configuration
- Clone the repository and navigate to the backend directory.
- The backend runs on port 3000. Set up a reverse proxy using nginx to ensure smooth deployment on EC2.
- Update the .env file to incorporate database connection details and port information.
### Solution
The steps to task 1 solution are as follows:
1. The docker running containers are as follows:
   ![image](https://github.com/user-attachments/assets/11ebe1fd-7877-4fbb-b3a7-b56b89777b58)
2. The Nginx configuration is as follows:
    ```
    server {
        listen 80;
        server_name app.vishcodes.live vish-elb-187455200.us-west-2.elb.amazonaws.com;
    
        location / {
            proxy_pass http://localhost:3000/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    
        # Serve static assets (JavaScript, CSS, etc.) from the frontend container
        location /static/ {
            proxy_pass http://localhost:3000/static/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    
        location /backend/ {
            proxy_pass http://localhost:3001/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
    
            # Disable caching for this location
            proxy_buffering off;
        }
    }

    ```
3. The .env is included on the fly in the pipeline. The code for the pipeline to include .env variables is as follows:
```
name: Build and Publish Docker Image

on:
  push:
    branches:
      - main

jobs:
  build_and_publish_backend:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_TOKEN }}

    - name: Create .env file
      run: |
        echo "MONGO_URI=${{ secrets.MONGO_URI }}" > backend/.env
        echo "PORT=${{ secrets.PORT }}" >> backend/.env

    - name: Build and push Docker image
      run: |
        docker buildx build --platform linux/amd64,linux/arm64/v8 --push --tag ${{ secrets.DOCKER_USERNAME }}/travel-memory-devops7:latest ./backend

  build_and_publish_frontend:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_TOKEN }}

    - name: Build and push Docker image
      run: |
        docker buildx build --platform linux/amd64,linux/arm64/v8 --push --tag ${{ secrets.DOCKER_USERNAME }}/travel-memory-devops7-frontend:latest ./frontend
    

```

## Task 2 - Frontend and Backend Connection
- Navigate to the `urls.js` in the frontend directory.
- Update the file to ensure the front end communicates effectively with the backend.
### Solution
1. The url.js with updated frontend looks like this:
```
export const baseUrl = "http://app.vishcodes.live/backend"
```
Notice that it points to my domain which points to the ELB with reverse proxy for `/backend` to talk to backend docker running on point `3001` mentioned in the prior step nginx configuration.
2. The screenshots displaying that the application is effectively talking to backend is as follows:
   - Homepage showing trip data:
      ![image](https://github.com/user-attachments/assets/820f424f-d035-4e65-88c6-4a2533af7741)

   - Specific trip data page:
      ![image](https://github.com/user-attachments/assets/dfb7ffbb-4bf7-409b-912b-139667c13021)

   - Backend health check:
      ![image](https://github.com/user-attachments/assets/ee7e1b2d-250c-4a9b-9113-151d8a9a6f85)

   - Backend GET request check:
      ![image](https://github.com/user-attachments/assets/e13e9d02-cb9b-4074-9d72-7d7dec083364)


## Task 3 - Scaling the Application
- Create multiple instances of both the frontend and backend servers.
- Add these instances to a load balancer to ensure efficient distribution of incoming traffic.
### Solution


## Task 4 - Domain Setup with Cloudflare
- Connect your custom domain to the application using Cloudflare.
- Create a CNAME record pointing to the load balancer endpoint.
- Set up an A record with the IP address of the EC2 instance hosting the front end.
### Solution



## Task 5 - Documentation
- Prepare comprehensive documentation detailing each step of the deployment process. Include relevant screenshots to make the process clear and reproducible.
- Design a deployment architecture diagram using [draw.io](https://www.draw.io/) to visualize the flow and connections.
### Solution

