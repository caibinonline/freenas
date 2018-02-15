pipeline {
  agent none

  environment {
    GH_ORG = 'freenas'
    GH_REPO = 'freenas'
  }
  stages {

    stage('Queued') {
      agent {
        label 'JenkinsMaster'
      }
      steps {
        echo "Build queued"
      }
    }

    stage('ixbuild') {
      agent {
        label 'FreeNAS-ISO'
      }
      post {
        success {
          archiveArtifacts artifacts: 'artifacts/**', fingerprint: true
          junit 'results/**'
	  stash includes: 'artifacts/iso/*.iso', name 'iso' (1)
        }
        failure {
          echo 'Saving failed artifacts...'
          archiveArtifacts artifacts: 'artifacts/**', fingerprint: true
        }
      }
      steps {
        checkout scm
        echo 'Starting iXBuild Framework pipeline'
        sh '/ixbuild/jenkins.sh freenas freenas-pipeline'
      }
    }

    stage('API testing') {
      agent {
        label 'FreeNAS-QA'
      }
      post {
        always {
          junit 'tests/results/*.xml'
        }
      }
      steps {
        echo 'Starting QA API testing'
        unstash 'iso'
        sh 'rm -rf tests/iso'
        sh 'mkdir -p tests/iso'
        sh 'mv artifacts/iso/*.iso tests/iso'
        sh 'ixautomation --run api-tests --systype freenas'
      }
    }
  }
}
