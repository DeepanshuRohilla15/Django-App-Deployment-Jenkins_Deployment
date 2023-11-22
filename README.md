# Django-App-Deployment
**DevOps Project of Django App Deployment on EC2 using Docker and Jenkins.**

## Resources &Technology Used
1. AWS Free Tier Account
2. EC2 Instance : t2 micro 
3. Django Todo App 
3. Docker Image
4. Jenkins 

## Project Flow
![This is an image](https://github.com/amitgitz/Django-App-Deployment/blob/Jenkins_Deployment/Project-Flow.svg)

## Step 1 : Running the App Locally
Before deploying the app on EC2 Instance , we will test run the app locally.

Create a folder for the project, open the folder in VS code and clone the app in the terminal 

Clone the repository.
```
git clone https://github.com/shreys7/django-todo.git
```

Creating Virtual Environment
```
virtualenv -p python3.11 env
```
Activate the vritual environment
In windows VS Code
``` 
env\Scripts\activate
```
Change directory to the app directory
  ``` 
  cd django-todo
  ```
create all the migrations to run this App.
```
python manage.py makemigrations
```
Apply this migrated command to run the following command
``` 
python manage.py migrate
```
Now we need to create the superuser
```
python manage.py createsuperuser
```
We just need to start the local Server
```
   python manage.py runserver
```
Now we can access the webapp on  http://127.0.0.1:8000/todos 

## step 2 : Create requirement file
We will requirement.txt file to freeze the dependencies requried to run the application
```
  pip freeze > requirement.txt
 ```
 
## step 3 : Set up  of AWS EC2 instance
1. Create the ec2 instance with following details
2. Name : Django-Application
3. AMI : Ubuntu 18 LTS
4. Key Pair : Lap_Connect
5. SG : Allowing SSH at Port 22

Make connection with EC2 Instance using git bash CLI with the help of key pair 

Now we are on EC2 instance. so create a new directory named Projects
```
mkdir projects
```
Now change directory to the newly created directory using cd command
```
cd projects
```
clone the django todo app repo in the project directory
```
git clone https://github.com/shreys7/django-todo.git
```
Change directory and move to the django-todo directory
```
  cd django-todo/
 ```
Add your instance's public IP to allowed host in setting.py file
```
   vi todoApp/settings.py
 ```
find allowed host and enter your public IP or just enter '*' so that all Ip are allowed

Now go back to terminal and Install docker on the EC2 instance
   ```sudo apt install docker.io
   ```
## step 4 : Create dockerfile and run it using vim create a Dockerfile
   vi Dockerfile
enter insert mode by pressing "i" and write the file as per the following
```
FROM python:
RUN pip install django==3.2


COPY . .


RUN python manage.py migrate


CMD ["python","manage.py","runserver","0.0.0.0:8001"]

```
press esc and use :wq and enter to save the file and exit vim.

Now build the docker file we created
```
sudo docker build . -t todo-app
 ````
   
Now you will  get a container ID. Copy that container ID and run the container with the following command.
``` 
sudo docker run -p 8001:8001 62761281ad8f
```

Now go to your browser and copy your public IP and use it to check if the app is running as shown in screenshot
enter your public IP:8001. example : 5.88.168.176:8001



## step 5 : Install and setup jenkins on your EC2 instance
Update your system in EC2 Terminal
```
   sudo apt update
   ```
Install java
```
   sudo apt install openjdk-11-jre
   ```
Install jenkins : Just copy these commands, paste them and run them one by one onto your terminal
```
   curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee \   /usr/share/keyrings/jenkins-keyring.asc > /dev/null
   echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \   https://pkg.jenkins.io/debian binary/ | sudo tee \   /etc/apt/sources.list.d/jenkins.list > /dev/null
 ```
 ```
   sudo apt-get update
   sudo apt-get install jenkins
   ```
Start Jenkins with these commands
```
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
   sudo systemctl status jenkins
   ```
Add port 8080 to your inbound rules in secuirity groups to allow traffic on it like we did for port 8001.

Now open Jenkins in browser by using public IP and port number



Get the password from given location and paste it here
```
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
 ```


Click install suggested plugin and go ahead ith the installation


create and setup your admin user and your are done. You must be on the Jenkins homepage now


Open your terminal and add jenkins to sudoers s shon in the screen shot so that when we build our job in jenkins it can have sudo access
   sudo visudo
use above command to open the file and then add jenkins like this



## Step 6 : create a GitHub repo for the project and push code from local to the repository
create a new GitHub repository with name you want.

copy the repository url, now go back to instance terminal and change the remote repository to your new repository

   git remote set-url origin https://github.com/deepanshurohilla15/jenkins-Deployment-projects.git
   
Add ll files to staged
```
   git add .
   ```
commit all the files
```
   git commit -m "added server code "
   ```
push the code to repository
```
   git push origin develop
   ```
## step 7 : Integrate jenkins with GitHub
Open your instance's jenkins and go to manage jenkins > configure sytsem > find github servers here and add your github credentials


save it.
## Step 8 : Deploy the app using Jenkins now
Create a new job as a freestyle project named todo-app

Then in source code management choose git. And paste your repositpory url there.



And in branches to build, write develop as we have pushed our code to the develop branch.


Now in build step. Add build step as eecute shell and write the following commands.
   sudo docker build . -t todo-app
   sudo docker run -p 8001:8001 -d todo-app


save it and click on build now to run the job.

Great your web app is running and Our Project is Completed with that.
