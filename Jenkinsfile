// pipeline {
//   agent any
//   tools { maven 'Maven-3' jdk 'OpenJDK-21' }
//   options { timestamps(); buildDiscarder(logRotator(numToKeepStr:'10')) }
//   stages {
//     stage('Checkout') { steps { checkout scm } }
//     stage('Build') { steps { sh 'mvn -B -DskipTests clean package' } }
//     stage('Test') {
//       steps { sh 'mvn -B test' ; junit 'target/surefire-reports/*.xml' }
//     }
//     stage('Archive') { steps { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true } }
//   }
//   post { always { cleanWs() } }
// }

pipeline {
  agent any

  environment {
    PYTHON = 'python3'
    VENV_DIR = 'venv'
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'ls -la'               // helpful to see workspace contents in console
      }
    }

    stage('Setup Python') {
      steps {
        // create venv and install requirements if file exists
        sh """
          ${env.PYTHON} --version || exit 1
          if [ -f requirements.txt ]; then
            ${env.PYTHON} -m venv ${env.VENV_DIR}
            . ${env.VENV_DIR}/bin/activate
            pip install --upgrade pip
            pip install -r requirements.txt
          else
            echo "requirements.txt not found — running without venv install"
          fi
        """
      }
    }

    stage('Run Tests') {
      steps {
        // run pytest and save junit-style xml
        sh """
          if [ -d ${env.VENV_DIR} ]; then
            . ${env.VENV_DIR}/bin/activate
          fi
          # ensure pytest is available (if not, show message)
          pytest -q --junitxml=pytest-results.xml || true
        """
        // publish JUnit-style results (if pytest produced file)
        junit allowEmptyResults: true, testResults: 'pytest-results.xml'
      }
    }

    stage('Archive') {
      steps {
        archiveArtifacts artifacts: '**/*.xml, **/*.log, **/*.txt', allowEmptyArchive: true
      }
    }
  }

  post {
    success { echo "Pipeline succeeded" }
    failure {
      echo "Pipeline failed — check console output"
    }
    always {
      cleanWs()
    }
  }
}
