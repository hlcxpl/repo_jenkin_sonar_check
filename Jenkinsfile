pipeline {
  agent any
  environment {
    SONAR_TOKEN = credentials('sonar-token')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install & Test') {
      agent {
        docker {
          image 'node:18'
          args '-v $HOME/.npm:/root/.npm'
        }
      }
      steps {
        sh 'npm install'
        sh 'npm test || true'
      }
    }

    stage('Dependency-Check') {
      steps {
        sh '''
        mkdir -p reports
        docker run --rm \
          -v "$PWD":/src \
          -v "$PWD/reports":/report \
          owasp/dependency-check:8.4.0 \
            --project "SafeNotes" \
            --scan /src \
            --format JSON,HTML \
            --out /report
        '''
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('sonar') {
          sh 'npx sonarqube-scanner'
        }
      }
    }
  }

  post {
    success { echo "✅ Análisis completo con OWASP + SonarQube" }
    failure { echo "❌ Algo falló en el proceso" }
  }
}
