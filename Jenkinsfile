pipeline{

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: acavaleiro/jenkins-nodo-java-bootcamp:1.0 
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
        '''
            defaultContainer 'shell'
        }
    }

  environment {
    registryCredential='dockerhup'
    registryFrontend = 'acavaleiro/spring-boot-app'
    sh 'java --version'
     
  }
  stages {
  stage("Builds"){
        steps{
           script{
            sh "mvn clean package -DskipTests"
                }
        }
    }

    stage('Push Image to Docker Hub') {
      steps {
        script {
          dockerImage = docker.build registryFrontend + ":$BUILD_NUMBER"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }

    stage('Push Image latest to Docker Hub') {
      steps {
        script {
          dockerImage = docker.build registryFrontend + ":latest"
          docker.withRegistry( '', registryCredential) {
            dockerImage.push()
          }
        }
      }
    }
  stage("Code Promotion"){
     steps {
      script{
   echo 'Quitar el sufijo snapShot, archivo Pom xml'
            }
          }
  }
  stage("Compile"){
    steps{
      script{
  echo 'compilar la aplicación Java'
            }
         }
  }
  stage("Unit Tests"){
    steps{
      script{
   echo'Lanzar los test unitarios'
            }
    }
  }
  stage("JaCoCo Tests"){
    steps{
      script{
   echo 'Test Jacoco'
            }
        }
  }
  stage("Quality Tests SonarQ"){
    steps{
       script{
    echo 'Pasar las quality Test app'
             }
         }
  }
  stage("Package .Jar"){
    steps{
        script{
     echo'Generar imagen app'
              }
         }
  }
  stage("Build Push Kaniko"){
    steps{
      script{
   echo 'Construir la imagen Spring'
            }
         }
  }
  stage("Run test environment"){
    steps {
        script{
   echo'iniciar el contenedo con el nodo y la aplicacion app Spring'
              }
          }
  }
  stage("Performance Test JMeter"){
    steps{
         script{
   echo'Lanzar los test con JMeter Newman'
               }
         }
  }
  stage("Nexus"){
    steps{
      script{
   echo'Guardar el artefacto al repositorio de Nexus'
            }
         }
  }
   stage("Deploy"){
    steps{
      script{
   echo'Realizar la subida imagen'
            }
        }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
