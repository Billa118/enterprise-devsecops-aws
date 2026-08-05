# Day 3 - Docker and Jenkins Setup

## Objective
Install Docker, Docker Compose, Java 21, and Jenkins on the AWS EC2 instance.

## Tasks Completed

- Connected to Ubuntu EC2 using SSH
- Updated Ubuntu packages
- Verified Git installation
- Added Docker official repository
- Installed Docker Engine
- Installed Docker Compose
- Added ubuntu user to Docker group
- Verified Docker using hello-world container
- Installed OpenJDK 21
- Added Jenkins repository
- Fixed Jenkins GPG key issue caused by the new signing key
- Installed Jenkins
- Enabled Jenkins service
- Started Jenkins service
- Accessed Jenkins using Public IP
- Installed Suggested Plugins
- Created Jenkins Admin User
- Logged into Jenkins Dashboard successfully

## Commands Used

```bash
sudo apt update
sudo apt install git -y

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo usermod -aG docker ubuntu
newgrp docker

docker run hello-world

sudo apt install openjdk-21-jdk -y

sudo apt install jenkins -y

sudo systemctl enable jenkins
sudo systemctl start jenkins

sudo systemctl status jenkins