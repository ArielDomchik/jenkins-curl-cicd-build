pipeline  {
    agent {
        label 'docker-agent'
    }
    stages {
        stage('Clone and Build CURL') {
            steps {
                sh 'bash scripts/build_and_test.sh'
            }
         }
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'curl/src/curl', fingerprint: true
            }
        }
    }

    post {
        success { 
            echo "Build and tests succeeded!"
            }
        failure {
            echo "Build or tests failed!"
        }
    }
}
