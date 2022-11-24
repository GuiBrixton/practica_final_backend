pipeline {

agent any

options {

buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')

}
stages {
stage('Hello Backend nuevo') {
steps {
sh '''
java -version
'''
      }
              }
      }

}