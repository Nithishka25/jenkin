pipeline {
  agent any
  tools { maven 'Maven-3' jdk 'OpenJDK-11' }
  options { timestamps(); buildDiscarder(logRotator(numToKeepStr:'10')) }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build') { steps { sh 'mvn -B -DskipTests clean package' } }
    stage('Test') {
      steps { sh 'mvn -B test' ; junit 'target/surefire-reports/*.xml' }
    }
    stage('Archive') { steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true } }
  }
  post { always { cleanWs() } }
}
