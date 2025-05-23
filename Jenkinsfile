pipeline  {
    agent any


    environment {
        REPO_DIR = '/workspace/curl'
        REPO_URL = 'https://github.com/curl/curl.git'
    }


    stages {
        stage('Fetch source from github (WAN)') {
            agent { label 'master' }
            steps {
		sh '''
                    rm -rf ${REPO_DIR}
                    git clone --depth 1 ${REPO_URL} ${REPO_DIR}
                '''
            }
         }



    
   stage('Build in isoloted agent') {
       	  agent { label 'docker-agent' }
            steps {
                sh '''
		    cd ${REPO_DIR}
                    ./buildconf
                    ./configure --with-openssl
                    make -j$(nproc)
		'''
            }
        }
    



	stage('Test in isolated agent') {
	  agent { label 'docker-agent' }
	    steps {
	      sh '''
                  cd ${REPO_DIR}
                  make test
       	       '''
        }
    }


     


	stage('Archive executable') {
	  agent { label 'docker-agent' }
	    steps {
	      sh '''
            cp ${REPO_DIR}/src/.libs/curl ./curl
          '''
	   	  archiveArtifacts artifacts: 'curl', fingerprint: true
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
