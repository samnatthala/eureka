
pipeline {
  agent {
    label 'k8s-slave'
  }
  tools {
    maven  'Maven3.8.8'
    jdk 'java17'
  }
  
  parameters {
    choice(name: 'sonarScans',
          choices: 'no\nyes',
          description: 'This will scan the applicaiton using sonar'
    )
    choice(name:'buildOnly',
          choices: 'no\nyes',
          description: 'This will only build the application'
    )
    choice(name: 'dockerPush',
            choices: 'no\nyes',
            description: "This will trigger the build, docker build and docker push"
        )
    choice(name: 'deployToDev',
            choices: 'no\nyes',
            description: "This will Deploy my app to Dev env"
        )
    choice(name: 'deployToTest',
            choices: 'no\nyes',
            description: "This will Deploy my app to Test env"
        )
    choice(name: 'deployToStage',
            choices: 'no\nyes',
            description: "This will Deploy my app to Stage env"
        )
    choice(name: 'deployToProd',
            choices: 'no\nyes',
            description: "This will Deploy my app to Prod env"
        )
  }

  environment {
    APPLICATION_NAME = "eureka"
    POM_VERSION = readMavenPom().getVersion()
    POM_PACKAGING = readMavenPom().getPackaging()
    DOCKER_HUB = "docker.io/dravikumar442277"
    DOCKER_CREDS = credentials('dravikumar442277_docker_creds')
    SONAR_URL = "http://34.132.67.4:9000/"
    SONAR_TOKENS = credentials('sonar_token')
  }

  stages {
    stage ('Building the application') { 
       when {
        anyOf {
            expression {
               params.buildOnly == 'yes'
               params.dockerPush == 'yes'
            } 
        }
       }
        steps {
            echo "this is ${env.APPLICATION_NAME} application"
            sh "mvn clean package -DskipTests=True"
        } 
    }
    stage ('unit test cases') {
      when {
        anyOf {
            expression {
               params.buildOnly == 'yes'
               params.dockerPush == 'yes'
            } 
        }
       } 
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
      when {
          anyOf {
                expression {
                        params.sonarScans == 'yes'
                        params.buildOnly == 'yes'
                        params.dockerPush == 'yes'
                    }
                }
            }
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
    /*stage ('Docker && Custom Format') {
     steps {
       //application name-version:
       echo "actual format: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
       // custom names for app jar
       // applicationname-buildnumber-branchnname-packaging
       echo "custm app: ${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}-${env.POM_PACKAGING}"
      }
    } */
    stage ('Docker Build') {
     when {
            anyOf {
                    expression {
                        params.dockerPush == 'yes'
                    }
                }
      } 
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
    stage ('Deploy to Dev') {
     when {
            anyOf {
                    expression {
                        params.deployToDev == 'yes'
                    }
                }
          } 
      steps {
       script {
        dockerDeploy ('dev','5761','8761').call()
       }
      }
    }
    stage ('Deploy to test') {
      when {
            anyOf {
                  expression {
                        params.deployToTest == 'yes'
                    }
                }
        }
      steps {
       script {
        dockerDeploy ('test','6761','8761').call()
       }
      }
    }
    stage ('Deploy to prod') {
          when {
                anyOf {
                    expression {
                        params.deployToProd == 'yes'
                    }
                }
            }
      steps {
       script {
        dockerDeploy ('prod','7761','8761').call()
       }
      }
    }
  }
}
// Define the dockerDeploy method outside the pipeline block
def dockerDeploy(envDeploy, hostPort, contPort) {
    return {
        echo "******************** Deploying to $envDeploy Environment ********************"
        stage ('Deploy to docker dev server') {
            script {
                withCredentials([usernamePassword(credentialsId: 'maha_creds_docker', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    // some block
                    sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker pull  ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                    try {
                        echo "***********stopping the container *********************************************************"
                        sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker stop  ${env.APPLICATION_NAME}-$envDeploy"
                        echo "**************** removing the container ****************************************************"
                        sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker rm  ${env.APPLICATION_NAME}-$envDeploy"
                    } catch (err) {
                        echo "caught the error: $err"
                    }
                    echo "********************** creating the container ****************************************"
                    sh "sshpass -p ${PASSWORD} -v ssh -o  StrictHostKeyChecking=no  ${USERNAME}@${docker_server_ip} docker run -d -p $hostPort:$contPort --name ${env.APPLICATION_NAME}-$envDeploy ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
                }
            }
        }
    }
}