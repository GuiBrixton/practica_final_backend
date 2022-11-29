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
    registryCredential='dockerhub'
    registryFrontend = 'acavaleiro/spring-boot-app'
   
    //**********************************************************************************
     NEXUS_VERSION = "nexus3"
     NEXUS_PROTOCOL = "http"
     NEXUS_URL = "192.168.49.3:8081"
     NEXUS_REPOSITORY = "Bootcamp"
     NEXUS_CREDENTIAL_ID = "CredencialNexus"
      DOCKERHUB_CREDENTIALS=credentials("dockerhub")
     DOCKER_IMAGE_NAME="acavaleiro/spring-boot-app"
    //**********************************************************************************
  }

  stages {

    stage("Prepare environment"){
        steps{
            sh "mvn -v"
            sh "java --version"
        }
    }
 //**************************************************POM-GIT************************************************************
     stage('A - Code Promotion') {
 
        when {
          branch 'main'
       }
        steps {
          script {
               // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
              pom = readMavenPom file: "pom.xml"
              POM_VERSION = ''
              version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
               echo "${version}"
                h "mvn versions:set -DremoveSnapshot=true"
               def versionsinsnapshot = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
              echo "${versionsinsnapshot}"
              sh "git add pom.xml"
              sh "git commit -m \"pom.xml update \""
              sh "git push https://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@github.com/GuiBrixton/practica_final_backend.git"
           }
         }
     }
 //**************************************************POM-GIT************************************************************
    stage("B - Compile"){
        steps{
            sh "mvn clean compile -DskipTests"
        }
    }

    stage("C - Unit Tests") {
        steps {
            sh "mvn test"
            junit "target/surefire-reports/*.xml"
        }
    }

    stage("D - JaCoCo Tests") {
        steps {
            jacoco()
          
        }
    }
    //**************************************************SONARQ************************************************************
     //stage('E - SonarQube analysis') {
     //    steps {
     //       withSonarQubeEnv(credentialsId: "	accesssonaQ", installationName: "SonarQ-server"){
     //           sh "mvn clean verify sonar:sonar -DskipTests"
    //         }
    //     }
    // }
    
    // stage('F - Quality Tests') {
    //   steps {

    //       withSonarQubeEnv(credentialsId: "accesssonaQ", installationName: "SonarQ-server"){
    //          sh "mvn clean verify sonar:sonar -DskipTests"
     //      }

     //      timeout(time: 2, unit: "MINUTES") {
     //          script {
     //             def qg = waitForQualityGate(webhookSecretId: 'sonarqube-credentials')
     //             if (qg.status != 'OK') {
     //                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
     //              }
     //         }
    //       }
    //   }
    // }
    //**************************************************************************************************************
    stage("G - Package"){
     
        steps{
            sh "mvn clean package -DskipTests"
        }
    }

       stage("H - kanico Build & Push"){
       steps{
           script {
            dockerImage = docker.build registryFrontend + ":$BUILD_NUMBER"
            docker.withRegistry( '', registryCredential) {
            dockerImage.push()
             }
            }
         }
    }
       stage(" I-  Run test environment"){
       steps{
           script {
          dockerImage = docker.build registryFrontend + ":latest"
          docker.withRegistry( '', registryCredential) {
          dockerImage.push()
          }
      }
  }
//*************************************************JMETER************************************************************
   stage("J - API Test o Performance Test"){
       steps{
           sh "Lanzar los test de JMeter o las pruebas de API con Newma"
       }
   }
   stage ("Setup Jmeter") {
          steps{
              script {
       
                  if(fileExists("jmeter-docker")){
                     sh 'rm -r jmeter-docker'
                  }
       
                  sh 'git clone https://github.com/GuiBrixton/jmeter-docker.git'
       
                   dir('jmeter-docker') {
       
                      if(fileExists("apache-jmeter-5.5.tgz")){
                          sh 'rm -r apache-jmeter-5.5.tgz'
                      }
                      sh 'apt-get install wget'
                      sh 'apt-get install python3-pip -y'                   
                      sh 'wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.5.tgz'
                      sh 'tar xvf apache-jmeter-5.5.tgz'
                      sh 'cp plugins/*.jar apache-jmeter-5.5/lib/ext'
                      sh 'mkdir test'
                      sh 'mkdir apache-jmeter-5.5/test'
                      sh 'cp ../src/main/resources/*.jmx apache-jmeter-5.5/test/'
                      sh 'chmod +775 ./build.sh && chmod +775 ./run.sh && chmod +775 ./entrypoint.sh'
                      sh 'rm -r apache-jmeter-5.5.tgz'
                      sh 'tar -czvf apache-jmeter-5.5.tgz apache-jmeter-5.5'
                      sh './build.sh'
                      sh 'rm -r apache-jmeter-5.5 && rm -r apache-jmeter-5.5.tgz'
                      sh 'cp ../src/main/resources/perform_test.jmx test'
                   }
              }
          }    
       }   
      stage ("Run Jmeter Performance Test") {
          steps{
              script {
                   dir('jmeter-docker') {
                      if(fileExists("apache-jmeter-5.5.tgz")){
                          sh 'rm -r apache-jmeter-5.5.tgz'
                      }
                      sh './run.sh -n -t test/perform_test.jmx -l test/perform_test.jtl'
                      sh 'docker cp jmeter:/home/jmeter/apache-jmeter-5.5/test/perform_test.jtl $(pwd)/test'
                      perfReport 'test/perform_test.jtl'
                   }      
              }
          }
       }
       stage ("Generate Taurus Report") {
           steps{
               script {
                       dir('jmeter-docker') {
                       sh 'pip install bzt'
                       sh 'export PATH=$PATH:/home/jenkins/.local/bin'
                       BlazeMeterTest: {
                           sh 'bzt test/perform_test.jtl -report'
                       }
                       }
               }
           }
       }
          stage("K - Nexus"){
       steps {
               script {
                  // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                  pom = readMavenPom file: "pom.xml"
                   // Find built artifact under target folder
                   filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                  // Print some info from the artifact found
                  echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                  // Extract the path from the File found
                   artifactPath = filesByGlob[0].path
                  // Assign to a boolean response verifying If the artifact name exists
                   artifactExists = fileExists artifactPath   
                       if(artifactExists) {
                       echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                       versionPom = "${pom.version}"                       
                          nexusArtifactUploader(
                          nexusVersion: NEXUS_VERSION,
                           protocol: NEXUS_PROTOCOL,
                         nexusUrl: NEXUS_URL,
                         groupId: pom.groupId,
                         version: pom.version,
                         repository: NEXUS_REPOSITORY,
                         credentialsId: NEXUS_CREDENTIAL_ID,
                         artifacts: [
                         // Artifact generated such as .jar, .ear and .war files.
                         [artifactId: pom.artifactId,
                         classifier: "",
                              file: artifactPath,
                              type: pom.packaging], // Lets upload the pom.xml file for additional information for Transitive dependencies
                             [artifactId: pom.artifactId,
                             classifier: "",
                               file: "pom.xml",
                               type: "pom"]
                           ]
                       )
                      } else {
                      error "*** File: ${artifactPath}, could not be found"
                   }
               }
           }
    }

    stage("L - Deploy"){
         steps{
        script {
          if(fileExists("configuracion")){
            sh 'rm -r configuracion'
          }
        }
        sh "git clone https://github.com/Guibrixton/kubernetes-helm-docker-config.git configuracion --branch test-implementation"
        sh "kubectl apply -f configuracion/kubernetes-deployment/spring-boot-app/manifest.yaml --kubeconfig=configuracion/kubernetes-config/config"
      }
    }

    }

  }

  post {
   always {
      cleanWs()
     sh 'docker logout'
   }
  }
}
