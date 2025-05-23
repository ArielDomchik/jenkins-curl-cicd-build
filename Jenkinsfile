pipeline {
    agent any

    environment {
        REPO_DIR = '/workspace/curl'
        REPO_URL = 'https://github.com/curl/curl.git'
    }

    stages {
        stage('Fetch source from GitHub (WAN)') {
            agent { label 'master' }
            steps {
                echo 'Cloning curl repository...'
                sh '''
                    set -e
                    rm -rf ${REPO_DIR}
                    git clone --depth 1 ${REPO_URL} ${REPO_DIR}
                '''
            }
        }

        stage('Build in isolated agent') {
            agent { label 'docker-agent' }
            steps {
                echo 'Starting build process...'
                sh '''
                    set -e
                    cd ${REPO_DIR}
                    echo "Running ./buildconf"
                    ./buildconf

                    echo "Configuring build with OpenSSL"
                    ./configure --with-openssl

                    echo "Building with make"
                    make -j$(nproc)
                '''
            }
        }

        stage('Test in isolated agent') {
            agent { label 'docker-agent' }
            steps {
                echo 'Running tests...'
                sh '''
                    set -e
                    cd ${REPO_DIR}
                    make test
                '''
            }
        }

        stage('Archive executable') {
            agent { label 'docker-agent' }
            steps {
                echo 'Preparing to archive curl binary...'
                sh '''
                    set -e

                    if [ ! -f ${REPO_DIR}/src/.libs/curl ]; then
                      echo "ERROR: curl binary not found!"
                      exit 1
                    fi

                    cp ${REPO_DIR}/src/.libs/curl ./curl

                    echo "Verifying binary type..."
                    file ./curl

                    echo "Checking binary runtime..."
                    LD_LIBRARY_PATH=${REPO_DIR}/lib/.libs ./curl --version || echo "Binary failed to run"
                '''
                archiveArtifacts artifacts: 'curl', fingerprint: true
            }
        }
    }

    post {
        success {
            echo '✅ Build and tests succeeded!'
        }
        failure {
            echo '❌ Build or tests failed! Check logs above.'
        }
    }
}

