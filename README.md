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

