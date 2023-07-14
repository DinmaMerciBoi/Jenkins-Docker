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
