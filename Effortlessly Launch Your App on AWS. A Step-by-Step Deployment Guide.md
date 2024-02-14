*2024-02-12*

[[technical]]

I have an personal website app built on nextjs want to deploy on production and run in docker. This article teach you deploy an docker container on AWS step by step.

## Create a EC2 VM
First, you need an AWS account. zzzz
when you login your account, you move to EC2 page create a VM in small demand.
I choose t4g.micro here. It's enough for my app.
![[Pasted image 20240211231200.png|500]]
### Connect to VM
you can connect your VM by dashboard of EC2 or ssh.
when you connect it, you can set up your environment.

## install dependencies
### Docker
([Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/))
1. Set up Docker's `apt` repository 
```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
2. install docker package
```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Git
```shell
sudo apt-get install git
```

### Clone your project from git
```shell
git clone <repository url>
```

### Container
Now, you can build your docker image and run in localhost, you can use curl to test
the sever whether run correctly.
```shell
# assume your dockerfile is in root folder of your project
docker build -t app .

# my app is run on 3000 port, you can change it if you want
docker run -dp 3000:3000 app

# check the server is running or not
curl localhost:3000
```

## Setup Nginx
Nginx is a web server that can also be used as a reverse proxy, load balancer, mail proxy, and HTTP cache. Nginx is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

### Install nginx
```sh
sudo apt-get update
sudo apt-get install nginx
```
  
### Configure nginx
Create a new configuration file for your application in the `/etc/nginx/sites-available` directory. Replace `yourdomain.com` with your actual domain name or IP address, and adjust the port number if your app runs on a different port.
*create file in sites-available*
```sh
sudo vim /etc/nginx/sites-available/app
```
*put these configuration to the file*
```nginx
   server {
       listen 80;
       server_name <your_ec2_ip or domain>;

       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
```

### Enable the configuration
Create a symbolic link to the configuration file in the `/etc/nginx/sites-enabled` directory to enable it.
```sh
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
```
### Test the nginx configuration
Test your Nginx configuration for syntax errors:
```sh
sudo nginx -t
```
If you see a message saying "syntax is okay" and "test is successful," your configuration is correct.
### Restart nginx
Restart Nginx to apply the changes:
```sh
sudo systemctl restart nginx
```
Done! Now, when you access `yourdomain.com`, Nginx will act as a reverse proxy and forward requests to your application running on port 3000. If you have an SSL certificate, you should also configure Nginx to handle HTTPS connections, which involves listening on port 443 and specifying the location of your SSL certificate and private key files.

## Use ECR Container Registry (Optional)
Sometimes your app's image is probably too large. You can use Amazon ECR (Elastic Container Registry) to store your image. Then, on your EC2 instance, pull the image from the container registry. This way, you don't need to build the image on your EC2 instance, which reduces computation on the VM.

following step you can use in CICD process.
### Create a repository
*in container registry page, click create repository*
*it is simple, just fill the name for your repository*
![[Pasted image 20240213121444.png|500]]
### Push your image
You can see the push commands when you click the *View push commands*
![[Pasted image 20240213121715.png|500]]
you can build image at your own computer, then push it into repository

## Pull image from VM
when you upload your image you can pull this image from your ec2 vm
```sh
docker pull <account_id>.dkr.ecr.<region>.amazonaws.com/<repository_name>:<tag>
# in my case -> docker pull 427243519225.dkr.ecr.ap-southeast-2.amazonaws.com/my_blog:latest
```
So you can use this image for your app. You don't even need to pull your code from GitHub
