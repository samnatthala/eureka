
pipeline {
  agent {
    label 'k8s-slave'
  }
  environment {
    APPLICATION_NAME = "eureka"
    POM_VERSION = readMavenPom().getVersion()
    POM_PACKAGING = readMavenPom().getPackaging()
    DOCKER_HUB = "docker.io/dravikumar442277"
    DOCEKR_CREDS = credentails('dravikumar442277_docker_creds')

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
       docker build  --force-rm --no-cache --pull --rm=true  -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}  ./.cicd
       docker images
       echo ************docker login now ********
       docker login ${DOCKER_HUB} -u ${CEKR_CREDS_USR} -p password ${CEKR_CREDS_PSW}

       """
    }

  }

}
}


