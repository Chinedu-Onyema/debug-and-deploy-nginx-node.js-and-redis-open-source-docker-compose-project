# Debug & Deploy a Load-Balanced Web Application with Docker Compose

This is a complete walkthrough for deploying a multi-container architecture featuring an NGINX reverse proxy load balancer, two replicated Node.js application instances, and a Redis cache database using Docker Compose. 

The deployment is managed via Docker Compose, a tool that facilitates starting and managing multiple Docker containers together using a single configuration file (compose.yml). Rather than creating this configuration from scratch, this walkthrough utilizes "Awesome Compose," an official Docker GitHub repository that serves as a "recipe book" containing ready-made Docker Compose examples for various application runtimes and technologies. Specifically, this project involves cloning and deploying the nginx-nodejs-redis repository from Awesome Compose.

A key aspect of this project from a DevOps perspective is understanding application structure to facilitate containerization, rather than deeply understanding specific code syntax. Furthermore, the walkthrough addresses a real-world scenario by identifying and fixing a specific bug within the cloned open-source repository.

Upon initial deployment (docker compose up), users may encounter a 502 Bad Gateway error because the web servers (web1 and web2) exit immediately after starting. The document identifies the root cause as a recently added security file (.npmrc) that prevents necessary scripts from running. This project includes steps to debug this issue by modifying the web/Dockerfile to start the application using a direct node command rather than an npm script, and subsequently rebuilding the containers without cache.

Final validation involves accessing the application via localhost:80 and refreshing the page to observe NGINX successfully distributing traffic between the two web server containers while Redis serves as the backend database.

### PDF GUIDE: [DEBUG AND DEPLOY OPEN SOURCE DOCKER PROJECT WITH DOCKER COMPOSE.pdf](https://github.com/user-attachments/files/32322045/DEGUB.AND.DEPLOY.OPEN.SOURCE.DOCKER.PROJECT.WITH.DOCKER.COMPOSE.pdf)

### WATCH VIDEO WALKTHROUGH HERE: https://youtu.be/eQ0LRdsP9x0


## PREREQUISITES

Docker Desktop installed and running.  
WSL2 (for Windows users) or a native Linux/macOS terminal.  
Git installed on your local environment


## DEPLOYMENT STEPS

### Step 1: System Check & Repository Setup

1) Open your terminal (WSL) and verify Docker and Git installations:

<PRE>docker --version</PRE>
<PRE>docker compose version</PRE>
<PRE>git --version</PRE>

(If Git is missing, install it using sudo apt update && sudo apt install git -y). 


2) Clone the official awesome-compose repository and navigate to the target sub-directory:

<PRE>git clone https://github.com/docker/awesome-compose.git</PRE>
<PRE>cd awesome-compose/nginx-nodejs-redis</PRE>



### Step 2: Initial Container Launch

1) Start all container services in foreground mode

<PRE>docker compose up</PRE>


2) Test the application in your browser by accessing http://localhost:80.
⚠️ Expected Issue: You may encounter a 502 Bad Gateway error.



### Step 3: Debugging the 502 Bad Gateway Bug

Cause of the Bug
A .npmrc file with ignore-scripts=true prevents npm start from executing application scripts properly.
As a result, the Node.js application containers (web1 and web2) crash immediately after starting, leaving NGINX without active upstream targets.

## Resolution Procedure

1) Open a second terminal window, navigate to the project directory, and inspect container statuses:

<PRE>cd awesome-compose/nginx-nodejs-redis</PRE>
<PRE>docker ps -a</PRE>

Notice that web1 and web2 show an EXITED (0) status while NGINX and Redis remain running.


2) Return to Terminal 1 and stop the stack by pressing Ctrl + C.


3) Update the Dockerfile inside the ./web directory to execute Node directly:

<PRE>nano web/Dockerfile</PRE>


4) Locate the last line containing CMD ["npm", "start"] and change it to:  

<PRE>CMD ["node", "server.js"]</PRE>


5) Save and exit the file (Ctrl + O, Enter, Ctrl + X).



### Step 4: Rebuilding & Verifying Load Balancing

1) Rebuild the web service images without using cache:

<PRE>docker compose build --no-cache web1 web2</PRE>


2) Start the full application stack again:

<PRE>docker compose up</PRE>


3) Open your web browser and navigate to http://localhost:80


4) Verify Behavior:

Load Balancing: Refresh the page multiple times. You will see the responding container name alternate between web1 and web2.

Redis Caching: The page visit counter increments across requests, confirming persistent caching via the Redis service.



## TEAR DOWN & CLEANUP

To stop containers and clean up built images and networks, run the following commands:

1) Stop and remove active containers

<PRE>docker compose down</PRE>


2) Remove containers along with all built service images:

<PRE>docker compose down --rmi all</PRE>


3) Confirm that all containers have been removed:

<PRE>docker compose ps -a</PRE>
