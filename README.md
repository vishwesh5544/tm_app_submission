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
1. The frontend and backend servers are contained accross 2 instances for test:
   ![image](https://github.com/user-attachments/assets/e5821363-d121-471e-9616-545a0d95c34d)
2. Added these instances to my target group. I have named my target group as - `vish-tg`:
   ![image](https://github.com/user-attachments/assets/32dcf783-d22c-41cc-9667-f3f5c0e8cffe)
3. My load balancer to point to Target group (vish-tg) is as follows:
   ![image](https://github.com/user-attachments/assets/39459b16-56c3-4f22-a843-84ba78d7039a)



## Task 4 - Domain Setup with Cloudflare
- Connect your custom domain to the application using Cloudflare.
- Create a CNAME record pointing to the load balancer endpoint.
- Set up an A record with the IP address of the EC2 instance hosting the front end.
### Solution
*I was unable to purchase a cloudflare domain or transfer my GoDaddy domain to cloudflare so I will be setting DNS via GoDaddy Panel.*
1. My GoDaddy domain DNS setting is as follows for adding ELB DNS to my domain records:
![image](https://github.com/user-attachments/assets/7174fe1b-2d8e-4f1b-9fee-257c6511a21c)
2. After successful DNS addition the domain is as follows:
![image](https://github.com/user-attachments/assets/1ba2b8a2-872a-485b-9c57-8f1ee78e1393)


## Task 5 - Documentation
- Prepare comprehensive documentation detailing each step of the deployment process. Include relevant screenshots to make the process clear and reproducible.
- Design a deployment architecture diagram using [draw.io](https://www.draw.io/) to visualize the flow and connections.
### Solution
- Architecture diagram using draw.io is as follows:
![tm-arch drawio](https://github.com/user-attachments/assets/7aabcd92-dbfb-4582-b597-096b6a99f871)
- The deployment process is as follows:
   1. Push to github `main` branch
   2. On trigger to `main` the workflow will run. The code to worflow is attached above.
   3. After the workflow succeeds, login to dockerhub and retrieve the tags
   4. SSH into ec2 and pull the docker images
   5. Run docker containers from images
   6. Make sure ngix config is correct and then test config using `sudo nginx -t`. If the Nginx tests pass, reload the server by `sudo systemctl restart nginx` or `sudo nginx -s reload`
- I have already provided my repo [fork](https://github.com/vishwesh5544/TravelMemory) above but the dockerfiles are as follows:
   - Frontend Dockerfile: 
      ```
      # Stage 1: Build the React app
      FROM node:18-alpine AS builder
      
      # Set the working directory
      WORKDIR /app
      
      # Copy the package.json and package-lock.json files
      COPY package*.json ./
      
      # Install the dependencies
      RUN npm install
      
      # Copy the rest of the application code
      COPY . .
      
      # Build the React application
      RUN npm run build
      
      # Stage 2: Serve the React app with Nginx
      FROM nginx:alpine
      
      # Copy the built React app from the builder stage to Nginx's default public directory
      COPY --from=builder /app/build /usr/share/nginx/html
      
      # Expose port 80 for the Nginx server
      EXPOSE 80
      
      # Start Nginx when the container launches
      CMD ["nginx", "-g", "daemon off;"]

      ``` 
      As you can see my frontend Dockerfile is multi stated Dockerfile where:
        1. In stage 1, I use the lighweight node alpine version and set alias as `builder` so I can use later. The stage 1 builds the React application.
        2. In the stage 2, from the `builder` context, build files are copied to Nginx static file deployment
   - Backend Dockerfile:
     ```
     FROM node:18-alpine

      WORKDIR /usr/src/app
      
      COPY package*.json ./
      
      RUN npm install
      
      COPY . .
      
      EXPOSE 3001
      
      CMD ["node", "index.js"]
     ```

## Additional Information
- Neovim Config: I use Neovim as my text editor. You can find my configuration [here](https://github.com/vishwesh5544/neovish).
- Operating System: I'm using Linux Pop!_OS for all my development work.
