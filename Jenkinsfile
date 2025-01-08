
pipeline {
  agent {
    label 'k8s-slave'
  }
  environment {
    APPLICATION_NAME = "eureka"
    POM_VERSION = readMavenPom().getVersion()
    POM_PACKAGING = readMavenPom().getPackging()

  }
  tools {
    maven  'Maven3.8.8'
    jdk 'java17'
  }
  stages {
    stage ('Building the application') {
        steps {
            echo "this is ${env.APPLICATION_NAME} application"
            sh "mvn clean package -DskipTests=True"
        }
    }
    stage ('unit test cases') {
     steps {
        echo "Performing Unit test cases for ${env.APPLICATION_NAME} application"
        sh "mvn test"  
     }
     post {
      always {
        junit 'target/surefire-reports/*.xml'
  }
}

  }  
  stage ('Docker && Custom Format') {
    steps {

    //application name-version:
    echo "actual format: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
   }
  }
}
}


