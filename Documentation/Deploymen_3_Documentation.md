<h1 align=center>Deployment 3 Documentation</h1>

# Deployment 3
- Any file with the .md extension will be formatted into a Markdown File

## Deployment Goal:
- Learn how to Deploy an application using 2 VPCs and a Jenkins Agent.

## Software/ Tools Used:
- VPC, EC2, Jenkins, GitHub, Nginx, Gunicorn, and Slack

## Virtual Private Clouds:
- Within this deployment I used 2 EC2s located in different VPCs.
- The fIrst EC2 is located in the default VPC created by Amazon. This EC2 housed the Jenkins server.
- The other EC2 was created within a different VPC I configured. I configured this EC2 to house the Jenkins agent in order to deploy my application.
- In an ideal setup I would have the Jenkins Server and Agent in the same VPC within different subnets, but the goal of this deployment is to communicate and connect 2 VPCs mirroring the work environment.

## EC2:

- I started off by launching the first Amazon EC2 Instance. I opened the standard Jenkins Ports 8080, 80, and 22.
- Once configured I ran Jenkins and set up a new node called awsDeploy, This node would SSH into my second EC2 to run the Jenkins agent and deploy my application
- In the second EC2 was created in the Public Subnet of my custom VPC. I opened ports 22 and 5000 for SSH access and 5000 for Nginx.
- The dependencies and pages needed for this EC2 were default-jre, python3-pip, python3.10-venv, and nginx

  

## Jenkins Agent/Nginx:
- I created a new node for the Jenkins Agent. After configuring the Agent I logged into my Virtual Machine to SSH int0 the Agent EC2.
- Once inside I needed to change some of the Nginx configuration. I ran:
```
 sudo nano /etc/nginx/sites-enabled/default
```
- Running that command allowed me to change the nginx default port from 80 to 5000 to later access my application. I also removed the existing location and added:

```
server {  
listen 5000;

location / {
proxy_pass http://127.0.0.1:8000;
proxy_set_header Host $host
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
}
```

## GitHub:

- To set up the Jenkins pipeline I needed to access the Jenkins file from the forked repository.
- I then added a Clean and Deploy stage to the Jenkins pipeline which would allow me to properly run gunicorn and the application.
```
stage('Clean') {
agent{label 'awsDeploy'}
steps{
sh '''#!/bin/bash
if [[ $(ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2) != 0 ]]
then
ps aux | grep -i "gunicorn" | tr -s " " | head -n 1 | cut -d " " -f 2 > pid.txt
kill $(cat pid.txt)
exit 0
fi

stage('Deploy') {
agent{label 'awsDeploy'}
steps {
keepRunning {
sh '''#!/bin/bash
pip install -r requirements.txt
pip install gunicorn
python3 -m gunicorn -w 4 application:app -b 0.0.0.0 --daemon
'''
```


## Jenkin Pipeline:

- After editing the Jenkins File, I constructed the Jenkins Pipeline. Initially I ran into problems during the testing and deploy stages but after some time I figured out the issues. I then got the pipeline to deploy.

![](https://lh5.googleusercontent.com/MG6IahVMTLhfrdB5t7qQKWY2XSq5FPsQCgZzE1nrhSf8OkzyKReatugaT8_bC4tzAHoUICTYZuEcJjAseEuhcYfbRBENR_7LjLwLtBc5nEHoZmXxaJN6TxX_dG8fa65qDDsovQrLcuQQxICRg6uCn3zEVff0IVgCKxz3M6RSk8TRnqPMGToeNOiYCQ)

  

## Slack:
- After I deployed my application I added a slack notification to notify me for any future pipeline changes.



## Diagram:

![](https://lh3.googleusercontent.com/2vnkPmtuahS73togM7hOJpEs1KfQUGNV-N48Eh0HRWHe6xmS5lQG6oBylx4FMauVIZDCVWAIDNDA7Q5ngMruETiWalPxlnon6-FXHra4KCruMdOw1dP1FhVUSxo0IT5gALZ-xSDbqseyOLARi_3KNDoQ3YRm7hFhOaNDxM4FWpaI5cL9NMK7wEGQog)

## Challenges:

- During this deployment I ran into a few challenges.

- Originally I ran into trouble trying to access the Agent EC2. At first I thought the IP wasn’t showing because It was placed in the Private subnet. After changing it to the public subnet I was still unable to see the public IP address. After reconfiguring the EC2 a third time I realized that I needed to enable Auto-assign public IP.

- The second challenge came during my Jenkins build configuration. I could not pass the testing stage of the build.

- After reading the errors I saw that my test wasn’t passing. I then went to the test_app.py file within github and fixed the test.

- My final challenge occurred when trying to get my application to appear. At the time my Jenkins Pipeline showed that the application was properly constructed. -After consulting with one of my peers we figured out the solution. Inorder for the changes made to the /etc/nginx/sites-enabled/default file to take effect nginx needed to be restarted. After running
```
sudo service nginx start
sudo service nginx stop
```
- The changes made to the nginx file took effect and my application appeared on port :5000.
