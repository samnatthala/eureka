pipeline {
    agent {
        label 'k8s-slave'
    }
    environment {
        APPLICATION_NAME = "eureka"
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = "docker.io/dravikumar442277"
        DOCKER_CREDS = credentials('dravikumar442277_docker_creds')
        SONAR_URL = "http://35.202.91.90:9000/"
        SONAR_TOKENS = credentials('sonar_token')
    }
    tools {
        maven 'Maven3.8.8'
        jdk 'java17'
    }
    stages {
        stage('Building the application') {
            steps {
                echo "Building ${env.APPLICATION_NAME}"
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Unit test cases') {
            steps {
                echo "Running unit tests for ${env.APPLICATION_NAME}"
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('SonarQube analysis') {
            steps {
                sh """
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=127-eureka \
                    -Dsonar.host.url=${env.SONAR_URL} \
                    -Dsonar.login=${env.SONAR_TOKENS}
                """
            }
        }
        stage('Docker Build') {
            steps {
                sh """
                    docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
                    docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                    docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}
                """
            }
        }
        stage('Deploy to Dev') {
            steps {
                script {
                    dockerDeploy('Dev', '5761', '8761').call()
                }
            }
        }
        stage('Deploy to Test') {
            steps {
                script {
                    dockerDeploy('Test', '6761', '8761').call()
                }
            }
        }
        stage('Deploy to Prod') {
            steps {
                script {
                    dockerDeploy('Prod', '7761', '8761').call()
                }
            }
        }
    }
}
def dockerDeploy(envDeploy, hostPort, conPort) {
    return {
        withCredentials([usernamePassword(credentialsId: 'maha_creds_docker', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
            script {
                sh "sshpass -p ${PASSWORD} ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                try {
                    echo "Stopping the container"
                    sh "sshpass -p ${PASSWORD} ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker stop ${env.APPLICATION_NAME}-${envDeploy}"
                    sh "sshpass -p ${PASSWORD} ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker rm ${env.APPLICATION_NAME}-${envDeploy}"
                } catch (err) {
                    echo "Error: $err"
                }
                echo "Creating the container"
                sh "sshpass -p ${PASSWORD} ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker run -d -p ${hostPort}:${conPort} --name ${env.APPLICATION_NAME}-${envDeploy} ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
            }
        }
    }
}
