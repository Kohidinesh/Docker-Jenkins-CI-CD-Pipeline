# Docker-Jenkins-CI-CD-Pipeline

Kindly follow the below blog for this requirment.
https://prakashkumar0301.medium.com/docker-jenkins-ci-cd-pipeline-dd54854125f3


![image](https://user-images.githubusercontent.com/121241275/213898449-3664ab35-cdff-4652-9b32-6b2b4e499ed5.png)

Jenkins-Docker Continuous Integration & Continuous Deployment Pipeline

1. Create Jenkins VM and install all the suggested plugins.

2. Under Available Manage Plugins section, search for docker, there are multiple Docker plugins, docker compose, docker build plugins. Please install all docker plugins.

3. Install Docker on Jenkins instance so that we can create docker images.

yum install docker

service docker start

systemctl enable docker

4. Configure JDK and Maven.

5. Since we will perform some operations such as checkout code-base and pushing an image to Docker Hub, we need to define the Docker Hub Credentials.

These definitions are performed under Jenkins Home Page -> Credentials ->Jenkins-> Global credentials -> Add Credentials menu and provide id and password and update ID as “dockerHubAccount”

6. Select Configure System to access the main Jenkins settings.

From Add a new cloud section select Docker from drop down .

Now we need docker host uri, the docker.sock file is owned by root and does not allow write permissions by other. You have to make it such that Jenkins can read/write to that socket file when mounted.

To enable the docker host URL you need to change permission on your Jenkins instance.

chmod 400 /var/run/docker.sock

Once you change permission then at dashboard, under Docker Host URI put

unix://var/run/docker.sock

And click on Test Connection, it will display installed docker and API version.

7. We have installed docker on Jenkins instance, now we are good to build docker images.

Create a Pipeline Jenkins job, you can name it as docker-ci-cd

Source Code URL: https://github.com/<github repo> and Jenkins file can be named as Jenkinsfile and save.

8. From Pipeline Syntax->select git under Sample Step. Provide all the details.

9. Go to Github repo and create a file called Jenkinsfile and paste Pipeline syntax content and commit your changes.

node{

stage (‘scm checkout’) {

git ‘https://github.com/<github repo>'

}

}

10. Save and build your job (just to verify)

11. In case of multiple branches and if you want to build through to any specific branch then

12. It’s time to build the job and create a package. From Pipeline syntax search for shell script, and

Create a script for maven package goal, and add script in Jenkinsfile file and build.

node{

stage (‘scm checkout’) {

git (‘https://github.com/<github repo>')

}

stage (‘Checkout to different branch’) {

sh “git branch -r”

sh “git checkout master”

}

stage (‘package stage’) {

sh label: ‘’, script: ‘mvn clean package ‘

}

}

13. We can define a new stage for docker image creation. Our docker images will contains Jenkins job’s artifact.

stage (‘docker image build’) {

sh ‘docker build -t pkw0301/prakash-app:1.0.0 .’

}

In order to copy Jenkins job artifact we have to write a Dockerfile, please keep it with your source code.

FROM tomcat:8

COPY /webapp/target/*.war /usr/local/tomcat/webapps/

14. Save and build job, ssh your Jenkins instance and verify Docker images.( If you get any permission denied error then please follow below commands and re-run your job)

chmod 400 /var/run/docker.sock

docker images

15. Push Docker image to Dockerhub/Registry. To do that we have to generate a script.

Pipeline syntax-> select withcredentials from drop down -> secret text → select dockerHubaccount -> generate script

Add stage to push docker images

stage (‘Push Docker image to DockerHub’) {

withCredentials([string(credentialsId: ‘dockerhubaccount’, variable: ‘dockerhubaccount’)]) {

sh “docker login -u pkw0301 -p ${dockerhubaccount}”

}

sh ‘docker push pkw0301/prakash-app:1.0.0’

}

Please provide your dockerhub id during login login command, so that you also can login and push images. Once you add stage, build you job. Verify docker images by login to dockerhub.

16. Now we have pushed docker images to dockerhub, In order to deploy these images to Development/QA/Prod environment, we need instaces

Create a instance for Development environment and install docker.

chmod 400 /var/run/docker.sock

From Pipeline syntax select sshagent->Add->Jenkins->

Kind: SSH Username with private key

ID: deploy-to-dev-docker

Username: <User ID>

Private key: please paste pem key of dev instance

Update your Jenkinsfile and add a new stage “deploy-to-dev”

stage (‘Deploy to Dev’) {

def dockerRun = ‘docker run -d -p 9000:8080 — name my-tomcat-app pkw0301/prakash-app:1.0.0’

sshagent([‘deploy-to-dev-docker’]) {

sh “ssh -o StrictHostKeyChecking=no <User Id>@<dev Instance IP> ${dockerRun}”

}

}

Save your changes and build.

You can access docker based application by the following URL.

<VM2 dev public ip>:9000/webapp
