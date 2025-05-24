pipeline {
  agent none

  environment {
    REPO_DIR = '/workspace/curl'
    REPO_URL = 'https://github.com/curl/curl.git'
  }

  stages {
    stage('Fetch source from GitHub (WAN)') {
      agent { label 'master' }
      steps {
        sh '''
          rm -rf ${REPO_DIR}
          git clone --depth 1 ${REPO_URL} ${REPO_DIR}
        '''
      }
    }

    stage('Build') {
      agent { label 'docker-agent' }
      steps {
        sh '''
          cd ${REPO_DIR}
          ./buildconf
          ./configure --with-openssl --enable-debug
          make -j$(nproc)
        '''
      }
    }

    stage('Test') {
      agent { label 'docker-agent' }
      steps {
        sh '''
          cd ${REPO_DIR}
          make test 2>&1 | tee ${WORKSPACE}/test-results.txt
        '''
        // Archive test results for viewing in Jenkins UI
        archiveArtifacts artifacts: 'test-results.txt', allowEmptyArchive: true
      }
    }

    stage('Archive Executable') {
      agent { label 'docker-agent' }
      steps {
        sh '''
          # Copy and strip the executable
          cp ${REPO_DIR}/src/.libs/curl ./curl
          strip ./curl
          
          # Create build info
          echo "Build Date: $(date)" > build-info.txt
          echo "Git Commit: $(cd ${REPO_DIR} && git rev-parse HEAD)" >> build-info.txt
          echo "Curl Version:" >> build-info.txt
          
          # Run curl with proper library path
          LD_LIBRARY_PATH=${REPO_DIR}/lib/.libs ./curl --version >> build-info.txt || echo "Binary failed to run" >> build-info.txt
        '''
        archiveArtifacts artifacts: 'curl,build-info.txt', fingerprint: true
      }
    }
  }

  post {
    success {
      echo "✅ Build and tests succeeded!"
    }
    failure {
      echo "❌ Build or tests failed!"
    }
    cleanup {
      node('docker-agent') {
        // Clean up workspace
        sh 'rm -rf ${REPO_DIR} || true'
      }
    }
  }
}
