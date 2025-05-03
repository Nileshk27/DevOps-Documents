# Project Setup using below tools

1) Maven
2) Git Hub
3) Jenkins
4) Docker

# Step-1 : Jenkins Server Setup in Linux VM #

1) Create Ubuntu VM using AWS EC2 (t2.medium) <br/>
2) Enable 8080 Port Number in Security Group Inbound Rules
3) Connect to VM using MobaXterm
4) Install Java

```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
```

3) Install Jenkins
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
4) Start Jenkins

```
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

5) Verify Jenkins

```
sudo systemctl status jenkins
```
	
6) Access jenkins server in browser using VM public ip

```
http://public-ip:8080/

```

7) Copy jenkins admin pwd

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
	   
8) Create Admin Account & Install Required Plugins in Jenkins

## Step-2 : Configure Maven as Global Tool in Jenkins ##

1) Manage Jenkins -> Tools -> Maven Installation -> Add maven <br/>

## Step-3 : Setup Docker in Jenkins ##
```
curl -fsSL get.docker.com | /bin/bash
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart jenkins
sudo docker version
```

## Create a script file:
nano setup_jenkins_nginx.sh
```
#!/bin/bash

echo "🚀 Installing NGINX..."
sudo apt update
sudo apt install nginx -y

echo "📁 Creating NGINX config for Jenkins..."
cat <<EOL | sudo tee /etc/nginx/sites-available/jenkins
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://localhost:8080;

        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;

        proxy_redirect off;
    }
}
EOL

echo "🔗 Enabling Jenkins site config..."
sudo ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/

echo "🧹 Removing default site config (optional)..."
sudo rm -f /etc/nginx/sites-enabled/default

echo "🔄 Restarting NGINX..."
sudo nginx -t && sudo systemctl reload nginx

echo "✅ NGINX reverse proxy setup complete!"
echo "🌐 Now access Jenkins via: http://<your-ec2-ip>/"

```
## Make it executable:
chmod +x setup_jenkins_nginx.sh

##Run it: 

./setup_jenkins_nginx.sh

# Step - 4 : Create Jenkins Job #

- **Stage-1 : Clone Git Repo** <br/> 
- **Stage-2 : Maven Build** <br/>
- **Stage-3 : Create Docker Image** <br/>
- **Stage-4 : Create Docker Container** <br/>
	
# Step - 5 : Trigger Jenkins Job #

# Step - 6 : Enable host port in security group inbound rules #

# Step - 7 : Access Application in Browser #

- **We should be able to access our application** <br/>

URL : http://public-ip:port/
	
# We are done with our Setup #
	
## Step - 8 : After your practise, delete resources we have used in AWS Cloud to avoid billing ##
