# GitHub Actions Self-Hosted Runnner Amazon Website CICD DevOps — Setup Guide


## For more projects, check out  
## [https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)
## [youtube-Link](https://www.youtube.com/@devopsHarishNShetty)

---
![img alt](https://github.com/harishnshetty/GitHub-Action-Amazon-DevSecOps/blob/bd54e7fd5d2ce24b50614b51ceb390dc6f860334/img.png)
---



---

## Table of Contents

- [Ports to Enable in Security Group](#ports-to-enable-in-security-group)
- [Prerequisites](#prerequisites)
- [System Update & Common Packages](#system-update--common-packages)
- [Docker](#docker)
- [Trivy (Vulnerability Scanner)](#trivy-vulnerability-scanner)
- [SonarQube (Docker)](#sonarqube-docker)
- [npm Installation](#npm-installation)



---

## Ports to Enable in Security Group

| Service    | Port  |
|------------|-------|
| HTTP       | 80    |
| SSH        | 22    |
| SonarQube  | 9000  |

---
## System Update & Common Packages

| Package               | Purpose                                         | Example                                                |
| --------------------- | ----------------------------------------------- | ------------------------------------------------------ |
| `bash-completion`     | Enables tab completion for commands and options | Type `git ch<Tab>` → completes to `git checkout`       |
| `wget`                | Downloads files from the internet               | `wget https://example.com/file.zip`                    |
| `git`                 | Version control system                          | `git clone`, `git commit`, `git push`                  |
| `zip`                 | Creates ZIP archives                            | `zip backup.zip file.txt`                              |
| `unzip`               | Extracts ZIP archives                           | `unzip backup.zip`                                     |
| `curl`                | Transfers data to/from URLs                     | Download files, call REST APIs                         |
| `jq`                  | Processes JSON from the command line            | `curl ... \| jq '.name'`                               |
| `net-tools`           | Legacy networking tools                         | `ifconfig`, `netstat`, `route`                         |
| `build-essential`     | Installs C/C++ build tools                      | `gcc`, `g++`, `make`                                   |
| `ca-certificates`     | Trusted SSL/TLS certificates                    | Lets HTTPS connections be verified                     |
| `apt-transport-https` | HTTPS support for APT on older Ubuntu versions  | Usually unnecessary on modern Ubuntu                   |
| `gnupg`               | GPG encryption and repository key management    | Import repository signing keys                         |
| `fontconfig`          | Font discovery and configuration                | Needed by Java apps, PDF generators, headless browsers |

bootstrap means inital setup/prepare of machine/begineer steps.
--
| Term                      | Simple Meaning         | Purpose                          | Example                                         |
| ------------------------- | ---------------------- | -------------------------------- | ----------------------------------------------- |
| **Bootstrap**             | Initial setup          | Prepare a system to perform work | Install `git`, `curl`, `docker` on a new server |
| **Boot**                  | Start a computer/OS    | Load the operating system        | Press the power button → Ubuntu starts          |
| **Provisioning**          | Create infrastructure  | Make servers, VMs, databases     | Create an AWS EC2 instance with Terraform       |
| **Configuration**         | Customize a system     | Set up installed software        | Configure Nginx, SSH, Docker                    |
| **Deployment**            | Release an application | Make an app available to users   | Deploy a React app to Nginx                     |
| **Initialization (Init)** | First-time setup       | Create initial data or state     | Create database tables on first run             |
| **Build**                 | Compile/package code   | Produce deployable artifacts     | `npm run build`, `mvn package`                  |
| **Install**               | Add software           | Make software available          | `sudo apt install git`                          |

Typical DevOps workflow
| Step         | What happens                    | Example                                  |
| ------------ | ------------------------------- | ---------------------------------------- |
| 1. Bootstrap | Install essential tools         | `git`, `curl`, `docker`, `jq`            |
| 2. Provision | Create infrastructure           | EC2, VM, Kubernetes cluster              |
| 3. Configure | Configure the infrastructure    | Install Nginx, configure firewall        |
| 4. Build     | Compile/package the application | `npm run build`                          |
| 5. Test      | Run automated tests             | `npm test`                               |
| 6. Deploy    | Publish the application         | Copy files to server or deploy container |
| 7. Monitor   | Observe the running application | Prometheus, Grafana, logs                |

Real-world
| Stage     | Command                            | Why                               |
| --------- | ---------------------------------- | --------------------------------- |
| Bootstrap | `sudo apt install git curl wget`   | Prepare the machine               |
| Provision | `terraform apply`                  | Create cloud resources            |
| Configure | `ansible-playbook setup.yml`       | Install and configure software    |
| Build     | `docker build -t myapp .`          | Create a Docker image             |
| Test      | `npm test`                         | Verify the application            |
| Deploy    | `kubectl apply -f deployment.yaml` | Run the application in Kubernetes |
| Monitor   | `kubectl logs`                     | Check application health          |

easy to remember
| Term          | Think of it as...              |
| ------------- | ------------------------------ |
| **Bootstrap** | Prepare the machine            |
| **Provision** | Create the machine             |
| **Configure** | Customize the machine          |
| **Build**     | Create the application package |
| **Deploy**    | Run the application            |
| **Monitor**   | Keep the application healthy   |
---
Bootstrap → Provision → Configure → Build → Test → Deploy → Monitor
----

```bash
sudo apt update
sudo apt upgrade -y

# Common tools
sudo apt install -y bash-completion wget git zip unzip curl jq net-tools build-essential ca-certificates apt-transport-https gnupg fontconfig
```

Reload bash completion if needed:
```bash
source /etc/bash_completion
```

**Install latest Git:**
```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git -y
```

---

## Docker

Official docs: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group (log out / in or newgrp to apply)
sudo usermod -aG docker $USER
newgrp docker
docker ps
```


Check Docker status:
```bash
sudo systemctl status docker
```

---

## Trivy (Vulnerability Scanner)
| Package               | Purpose                                                      | Example use                                        |
| --------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| `wget`                | Downloads files from the internet                            | Download a GPG key, installer, or script           |
| `apt-transport-https` | Allows APT to use HTTPS repositories (older Ubuntu versions) | Install packages from `https://...` repositories   |
| `gnupg`               | Verifies package signatures and manages GPG keys             | Import a repository's signing key                  |
| `lsb-release`         | Provides Ubuntu distribution information                     | Detect the Ubuntu codename like `jammy` or `noble` |

apt-transport-https not used in modern ubuntu.

Docs: [https://trivy.dev/v0.65/getting-started/installation/](https://trivy.dev/v0.65/getting-started/installation/)

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy


trivy --version
```

---

## SonarQube (Docker)

```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:lts-community
```

---

## npm Installation

Docs: [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# In lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 22

# Verify the Node.js version:
node -v # Should print "v22.18.0"
nvm current # Should print "v22.18.0"

# Verify npm version:
npm -v # Should print "10.9.3"
```



## Github Credentials to Store

| Purpose       | ID            | Type          | Notes                               |
|---------------|---------------|---------------|-------------------------------------|
| Email         | EMAIL_USER    | emailaddress@gmail.com|                                  |
| Email-app-Pass     | EMAIL_PASS   | Secret text   | From app password         |
| Docker-username    | DOCKER_USERNAME   | your-docker-id   | From your Docker Hub profile       |
| Docker-username    | DOCKER_PASSWORD   | token   | From your Docker Hub token       |
| sonar-qube    | follow the same step
