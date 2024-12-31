pipeline{
    agent any

    tools {
        maven "maven3"
        jdk "jdk17"
    }
    environment {
        SONAR_SCANNER = tool "sonar-scanner"
        DOCKER_USERNAME = "akash2147"
        DOCKER_IMAGE = "$DOCKER_USERNAME/snap:${env.BUILD_NUMBER}"
    }
    stages {

        stage('Cleanup workspace'){
            steps{
                cleanWs(deleteDirs: true, disableDeferredWipeout: true)
            }
        }
        stage('Checkout') {
            steps{
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/PrinceAkash007/SonarQube-Project-Kastro.git'
            }
        }

        stage('Maven compile') {
            steps {
                script {
                    sh "mvn clean compile"
                }
            }
        }

        stage('Maven test') {
            steps{
                sh "mvn test"
            }
        }

        stage('Sonar scanning'){
            steps{
                withSonarQubeEnv('sonar-scanner') {
                    sh "$SONAR_SCANNER/bin/sonar-scanner -Dsonar.projectName=youtube -Dsonar.projectKey=youtube -Dsonar.java.binaries=target"
                }
            }
        }

        stage('Maven package'){
            steps{
                script{
                    sh "mvn package"
                }
            }
        }

        stage('Docker Image build') {
            steps{
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Image push to docker'){
            steps{
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        
                        sh """
                            echo "Docker username: $DOCKERHUB_USERNAME"
                            echo "Docker image: $DOCKER_IMAGE"
                            docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD
                            docker push $DOCKER_IMAGE

                        """
                    }
                }
            }
        }

        stage('Docker container build'){
            steps{
                script{
                    // Stop any existing container with the same name
                    sh "docker stop snap || true && docker rm snap || true"
                    // Running the container 
                    sh "docker container run -d --name snap -p 5555:5555 $DOCKER_IMAGE"
                }
            }
        }
    }
}