pipeline {
  agent any

  tools {
    maven 'maven' // Asegúrate de que está configurado en Jenkins
  }

  environment {
    // SONAR_TOKEN se llena con el valor seguro de las credenciales
    SONAR_TOKEN = credentials('sonarcloud') 
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean install'
      }
    }

    stage('SonarCloud Analysis') {
      steps {
        withSonarQubeEnv('sq1') { // Este nombre debe coincidir con el configurado en Jenkins
          sh """
            mvn sonar:sonar \
              -Dsonar.login=$SONAR_TOKEN
          """
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 1, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }
}
