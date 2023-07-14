# Jenkins-Docker
Integrating Docker within Jenkins

## 1. Clone this Repository

This Repository contains a Dockerfile that builds a custom Jenkins image, with Docker installed within it.
* The `FROM` command tells Docker the base image (jenkins/jenkins:lts) from which to build the custom image.
* The first `RUN` command updates all essential packages using `apt-get update`. Then installs the new ones needed for the custom Jenkins image using `apt-get install`.
* The following `RUN` command downloads the Docker Linux Debian distribution CLI. Then the next one adds it to the repository.
* And the last `RUN` command adds Jenkins user to the Docker group.

## 2. Build the Custom Jenkins-Docker Image

With this command, build and tag the custom jenkins-docker image appropriately
```bash
docker build -t username/custom-jenkins-docker-image:v
```

## 3. Run the Container with the Image

With this command, run the container with the custom jenkins-docker image
```bash
docker run --name jenkins-docker -d -p 8080:8080 -p 50000:50000 -v /var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home username/custom-jenkins-docker-image:v
```

## 4. A few things to note

### Store Jenkins data

To store Jenkins, you need to create an explicit volume for Jenkins on your host machine. To do that, the argument `-v jenkins_home:/var/jenkins_home` was added when running the custom Jenkins-Docker image.

### Enabling use of Docker daemon in the Jenkins container

To do this, connect the Docker CLI in the Jenkins container to the Docker daemon on the host machine by bind mounting the daemon’s socket into the container with the `-v` flag. When running the image, this argument: `/var/run/docker.sock:/var/run/docker.sock` was added.

## 5. Configure your Jenkins Environment

* Access Jenkins UI on the web page
  `localhost:8080`
* Retrieve the initial password with this command
  ```bash
  docker exec -it jenkins-docker cat /var/jenkins_home/secrets/initialAdminPassword
  ```
* Install Suggested Plugins
* Create the First Admin User

## 6. A Simple Sample Pipeline Script

* Create a Pipeline project
* Paste this sample pipeline script
```bash
pipeline{
    agent any
     tools {
        maven "maven3.9.3"
    }
    
    stages{
        stage('1. Git clone'){
            steps{
                sh "echo 'Cloning the latest version of the application'"
                git "https://github.com/MerciBoi/maven-web-application"
            }
        }
        stage('2. Test and Build'){
            steps{
                sh "echo 'Running Unit testing'"
                sh "echo 'Unit testing done. Creating packages.'"
                sh "mvn clean package"
                sh "echo 'Artifacts created'"
            }
        }
        stage('3. Predeployment'){
            steps{
            withCredentials([usernamePassword(credentialsId: 'dockercred', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                sh "echo 'Predeployment in progress'"
                sh "docker build -t chydinma/test:$BUILD_NUMBER ."
                sh "docker login -u $DOCKER_USER -p $DOCKER_PASS"
                sh "docker push chydinma/test:$BUILD_NUMBER"
                }
            }
        }
        stage('4. Email Notification'){
            steps{
                sh "echo 'Application deployment successful'"
                sh "echo 'Congratulations, end-to-end automation process completed.'"
                emailext (attachLog: true, body: 'The build is complete.', subject: 'Build Complete', to: 'dimmalives@gmail.com')
                }
            }
        }
    }
```

### Before you run the build, ensure you do the following

* Install tool `Maven`, here used as `maven3.9.3`
* Create Credential for DockerHub login of type `Username and Password`, here used as `dockercred`
* Bind the credential `dockercred` to variables as used in the pipeline script
  * Go to `Pipeline syntax` generator
  * Use `withcredentials: Bind credentials to variables`
  * Bindings should be `Username and password (separated)`
  * Specify `Username Variable` and `Password Variable`, here used as `DOCKER_USER` and `DOCKER_PASS` respectively
  * Generate Pipeline Script
  * Copy and paste, as shown above
* Notice that Email Notification was configured, you may edit the script to remove that if you have not configured your Jenkins for Email Notification, or otherwise.

## 7. Now, Run your build!

## Hope the build is successful!

