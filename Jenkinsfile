
pipeline {
  agent {
    label 'k8s-slave'
  }
  environment {
    APPLICATION_NAME = "eureka"
  }
  tools {
    maven = "Mavenmy"
    java = "java17"
  }
  stages {
    stage ('Building the application') {
        steps {
            echo "this is ${env.APPLICATION_NAME} application"
            sh "mvn clean package"
        }
    }
  }  
}