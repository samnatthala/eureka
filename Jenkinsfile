
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
    SONAR_URL = "http://34.59.2.46:9000/"
    SONAR_TOKENS = credentials('sonar_token')
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
    stage ('Sonar stage now') {
      steps {
        sh """
           echo " Now started sonar code quality coverage stage now"
           mvn clean verify sonar:sonar \
            -Dsonar.projectKey=127-eureka \
            -Dsonar.host.url=${env.SONAR_URL} \
            -Dsonar.login=${env.SONAR_TOKENS}
        """
      }
    }
   
    stage ('Docker && Custom Format') {
     steps {

       //application name-version:
       echo "actual format: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
       // custom names for app jar
       // applicationname-buildnumber-branchnname-packaging
       echo "custm app: ${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}-${env.POM_PACKAGING}"
   }
  }
   stage ('Docker Build') {
    steps {
       sh """
       ls -la 
       ls -la ./.cicd
       cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}  ./.cicd
       ls -la ./.cicd
       echo ************docker build now working********
       docker build  --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}  -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}  ./.cicd
       docker images
       echo ************docker login now ********
       docker login  -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
       docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}

       """
    }
  }
  stage ('Deploy to dokcer dev server') {
    steps {
      echo "*****************Deploying to Dev Environment here########################"
      withCredentials([usernamePassword(credentialsId: 'maha_creds_docker', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
    // some block
    
     // sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} hostname "
      sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker pull  ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} "
      echo "***********stopping the container *********************************************************"
      sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker stop  ${env.APPLICATION_NAME}-dev"
      echo "**************** removing the container ****************************************************"
      sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker rm  ${env.APPLICATION_NAME}-dev"
      echo "********************** creating the container ****************************************"

      sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker run -d -p 5761:8761 --name ${env.APPLICATION_NAME}-dev ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
}
   
    }

  }

}
}

