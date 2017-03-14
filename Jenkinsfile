pipeline {
  agent { label 'docker' }

  stages {
    stage('Build') {
      agent {
        docker {
          image 'maven:3.3.9'
          reuseNode true
        }
      }
      steps {
        sh 'mvn -B clean package -DskipTests=true findbugs:findbugs'
      }
      post {
        success {
          step([$class: 'FindBugsPublisher', pattern: 'target/findbugsXml.xml', unHealthy: ''])
        }
      }
    }

    stage('Test') {
      agent {
        docker {
          image 'maven:3.3.9'
          reuseNode true
        }
      }
      when {
        branch '**/master'
      }
      steps {
        sh 'mvn -B clean package -Pintegration'
      }
      post {
        always {
          archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
        success {
          junit '**/target/surefire-reports/TEST-*.xml'
        }
        unstable {
          junit '**/target/surefire-reports/TEST-*.xml'
        }
      }
    }
    stage('Release Test') {
      agent {
        docker {
          image 'maven:3.3.9'
          reuseNode true
        }
      }
      when {
        branch '**/release-*'
      }
      steps {
        sh "mvn -B clean package -Prelease -Dversion='${env.BRANCH_NAME.drop(env.BRANCH_NAME.lastIndexOf('-')+1)}.$BUILD_NUMBER'"
      }
      post {
        always {
          archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
        success {
          junit '**/target/surefire-reports/TEST-*.xml'
        }
        unstable {
          junit '**/target/surefire-reports/TEST-*.xml'
        }
      }
    }
  }
}
