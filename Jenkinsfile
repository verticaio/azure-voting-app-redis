pipeline {
   agent { label 'master'}
   environment {
    PROJECT_NAME = 'myproject'
	 DOCKER_REPO = 'demodocker'
	 MS_NAME = 'vote_app'
    REGISTRY_URL = '192.168.0.106:8082'
	 DOCKER_REPO_PASSWORD = credentials('docker_api_token') 
    KUBETEST_CREDS = credentials('kubetest_config')
    FULL_PATH_BRANCH = "${sh(script:'git name-rev --name-only HEAD', returnStdout: true)}" 
    GIT_BRANCH = FULL_PATH_BRANCH.substring(FULL_PATH_BRANCH.lastIndexOf('/') + 1, FULL_PATH_BRANCH.length())
    }

   stages {
      stage('Verify Branch') {
         steps {
            echo "$GIT_BRANCH"
         }
      }

      stage('Docker Login') {
         steps {
               echo 'Login to nexus repo...'
               sh "docker login -u admin -p $DOCKER_REPO_PASSWORD $REGISTRY_URL/$DOCKER_REPO"
      }} 

      stage("Docker Build") {
         steps {
               sh "cd azure-vote/ ; docker build --rm -t $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP . ; cd .." 
      }}

      stage("Docker Push ") {
         steps {
               echo 'Start pushing to nexus repo...'
               sh  "docker push $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP"
               sh  "docker rmi $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP"
      }} 
      stage('Docker Build02') {
         steps {
            sh(script: 'docker images -a')
            sh(script: """
               cd azure-vote/
               docker images -a
               docker build -t jenkins-pipeline .
               docker images -a
               cd ..
            """)
         }
      }
      stage('Start test app') {
         steps {
            sh(script: """
               docker-compose up -d
               chmod +x ./scripts/test_container.sh
               ./scripts/test_container.sh
            """)
         }
         post {
            success {
               echo "App started successfully :)"
            }
            failure {
               echo "App failed to start :("
            }
         }
      }
      stage('Run Tests') {
         steps {
            sh(script: """
               pytest ./tests/test_sample.py
            """)
         }
      }
      stage('Stop test app') {
         steps {
            sh(script: """
               docker-compose down
            """)
         }
      }
   }
}
