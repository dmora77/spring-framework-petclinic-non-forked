pipeline {
  agent any

  tools {
    maven 'Maven_3' // Debes tener esta herramienta configurada en Jenkins
  }
  environment {
    SONAR_TOKEN = credentials('sonarcloud')         // Token para SonarCloud o SonarQube
    NVD_API_KEY = credentials('NVD_API_KEY_2')            // Token para Dependency Check
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
  }

  triggers {
    githubPush()
    // O si usas GitHub multibranch:
    // pollSCM('@daily')
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/dmora77/spring-framework-petclinic-non-forked.git', branch: 'main'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean install verify package'
      }
    }

    stage('Dependency Check') {
      steps {
        sh """
          dependency-check.sh \
            --project spring-framework-petclinic-non-forked \
            --format JSON \
            --disableNodeJS \
            --disableRetireJS \
            --nvdApiKey ${NVD_API_KEY} \
            --scan .
        """
        // Asegúrate de que dependency-check.sh esté instalado y accesible
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQubeLocal') {
          sh """
            mvn sonar:sonar \
              -Dsonar.login=${SONAR_TOKEN}
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

  post {
    success {
      echo 'Build completed successfully.'
    }
    failure {
      echo 'Build failed.'
    }
  }
}
