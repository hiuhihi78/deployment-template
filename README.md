
# Build and publish Application with Docker
## Overview
##### 🧭 System Architecture (FE + BE + Nginx)
    ┌──────────────────────┐           ┌──────────────────────┐
    │   FE (Docker Image)  │           │   BE (Docker Image)  │
    └──────────┬───────────┘           └──────────┬───────────┘
               │                                  │
               │                                  │
               │                                  │
    ┌──────────▼──────────┐           ┌──────────▼──────────┐
    │ Nginx (Docker - FE) │           │     Run CMD (BE)    │
    └──────────┬──────────┘           └──────────┬──────────┘
               │                                  │
               │                                  │
        FE:3000 (container)               BE:5000 (container)
               │                                  │
               │                                  │
               └───────────────┬──────────────────┘
                               │
                               │
                    ┌──────────▼──────────┐
                    │    Nginx (Host)     │
                    └──────────┬──────────┘
                               │
                ┌──────────────┴──────────────┐
                │                             │
                │                             │
                ▼                             ▼
           Frontend                       Backend API
                │                             │
                └──────────────┬──────────────┘
                               │
                               ▼
                            Client
 ### 📁 Project Structure
 ```
 project-root/  
│  
├── frontend/ # Frontend (React / Vue / ...)  
│ ├── Dockerfile # Build Docker image for FE  
│ ├── nginx.conf # Nginx config inside FE container  
│ └── ... # Source code  
│  
├── backend/ # Backend (.NET / Node / ...)  
│ ├── Dockerfile # Build Docker image for BE  
│ └── ... # Source code  
│  
├── nginx/ # Nginx (Host)  
│ ├── nginx.conf # Reverse proxy config (Host Nginx)  
│ └── ssl/ # SSL certificates  
│ ├── cert.pem  
│ └── key.pem  
│  
├── compose.yaml # Docker Compose (run multiple services)  
├── .env # Environment variables  
 ```
### Build and Config and Publish application
1. Setting config (Dockerfile, nginx config, docker compose)
2. Buid image FE, BE + Push Docker Hub
3. Settubg config in VM
    + Install Docker
    + Install nginx
    + Copy config file into VM (docker compose, ssl certificate, env)
    + Config port
4. Run docker compose
### Setting detail
#### 1. Setting config (Dockerfile, nginx config, docker compose)
Note: Sample

#### 2. Buid image FE, BE + Push Docker Hub)
##### Build image
```
docker build -t <image_name>:<tag> -f <folder>
docker tag <local_image> <dockerhub_username>/<repo_name>:<tag>
docker push <dockerhub_username>/<repo_name>:<tag>
```
##### Push image
```
docker push <dockerhub_username>/<repo_name>:<tag>
```
#### 2. Settubg config in VM
##### Install Docker
```
sudo apt remove docker docker-engine docker.io containerd runc

sudo apt update
sudo apt install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER
newgrp docker

docker version
docker compose version
```
##### Login Docker
```
docker login -u  (username)
(password)
```
##### Install nginx
```
sudo apt update
sudo apt install nginx -y

sudo ufw allow 'Nginx Full'

sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

##### Copy config file into VM (docker compose, ssl certificate, env)
######  Create project folder
```
mkdir (project-name)
cd (project-name)
```
######  Create Docker compose file
```
nano docker-compose.yml
```
######  Create config folder nginx
```
mkdir ngnix
cd ngnix
nano nginx.conf
```

######  Create config folder ssl
```
mkdir ssl
cd ssl
nano cert.pem
nano key.pem

sudo mkdir -p /etc/nginx/ssl
sudo cp /home/(username)/(project-foler)/nginx/ssl/cert.pem /etc/nginx/ssl
sudo cp /home/(username)/(project-foler)/nginx/ssl/key.pem /etc/nginx/ssl
sudo cp /home/(username)/(project-foler)/nginx/nginx.conf /etc/nginx/nginx.conf

sudo nginx -t

sudo systemctl reload nginx
sudo systemctl stop nginx 
sudo systemctl start nginx 
```
#### 4. Run docker compose
```
docker compose -f docker-compose.yml up -d

=== other command ===
docker compose -f docker-compose.yml down
docker image prune
docker image prune -a
docker rmi -f $(docker images -q)
```

#### Linux command sample
#### Linux Basic Commands

```bash
# 📂 Directory & Navigation
pwd                 # Show current directory
ls                  # List files and folders
ls -l               # List with details
cd /path/to/folder   # Change directory
cd ..               # Go up one directory
mkdir folder_name    # Create a new directory
rmdir folder_name    # Remove an empty directory

# 📄 File Operations
touch file.txt       # Create an empty file
cp source dest       # Copy file or folder
mv source dest       # Move or rename file/folder
rm file.txt          # Delete a file
rm -r folder_name    # Delete folder recursively
cat file.txt         # Show file content
less file.txt        # View file content page by page
nano file.txt        # Edit file in terminal

# 🔍 Search & Info
find . -name "file*"  # Search files by name
grep "text" file.txt  # Search text inside file
wc -l file.txt        # Count lines
df -h                 # Disk usage
du -sh folder_name    # Folder size

# 🔧 Permissions
chmod 755 file.txt     # Change file permissions
chown user:group file  # Change file owner

# 🖥 System Info
whoami                 # Show current user
uname -a               # Kernel info
top                    # Real-time processes
ps aux                 # Show running processes

# 🐳 Docker Basic
docker ps              # List running containers
docker ps -a           # List all containers
docker images          # List local images
docker stop <id>       # Stop container
docker rm <id>         # Remove container
docker rmi <image>     # Remove image
