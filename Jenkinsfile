pipeline {
   agent { any }

   environment {
    PROJECT_NAME = 'myproject'
	 DOCKER_REPO = 'demodocker'
	 MS_NAME = 'vote-app'
    REGISTRY_URL = 'nexus:8082'
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
               dir("$WORKSPACE/azure-vote"){
               sh "docker build --rm -t $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP ." 
      }}}

      stage("Docker Push ") {
         steps {
               echo 'Start pushing to nexus repo...'
               sh  "docker push $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP"
      }} 

      // stage("Docker REMOVE IMAGE LOCAL ") {
      //    steps {
      //          echo 'Start removing image from local env...'
      //          sh  "docker rmi $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP"
      // }} 

      stage('Docker Build Different') {
         steps {
            sh(script: 'docker images -a')
            sh(script: """
               cd azure-vote/
               docker build -t jenkins-pipeline .
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

      stage('Container Scanning In Paralel') {
         parallel {
            stage('Run OTHER Tool') {
               steps {
                  sh(script: """
                     echo "RUN another security scanning here clair, fortify, anchore and etc"
                  """)
               }
            }
            stage('Run Trivy') {
               steps {
                  sleep(time: 30, unit: 'SECONDS')
                   sh(script: """
                   trivy    image  --exit-code 0 --severity MEDIUM,LOW  $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP # continue when find it MEDIUM,LOW 
                   #trivy   image  --exit-code 1 --severity CRITICAL,HIGH $REGISTRY_URL/$DOCKER_REPO/$PROJECT_NAME/$MS_NAME:$BUILD_TIMESTAMP # stop pipeline when find it CRITICAL,HIGH 
                   """)
               }
            }
         }
      }
      stage('Approve PROD Deploy') {
         when {
            branch 'master'
         }
         options {
            timeout(time: 1, unit: 'HOURS') 
         }
         steps {
            input message: "Deploy?"
         }
         post {
            success {
               echo "Production Deploy Approved"
            }
            aborted {
               echo "Production Deploy Denied"
            }
         }
      }

      stage('Deploy to PROD') {
         when {
            branch 'master'
         }
         environment {
            ENVIRONMENT = 'prod'
         }
         steps {
            echo "Deploying to ${ENVIRONMENT}"
            sh(script: """
            sed -i  "s/ms-name/$MS_NAME/g ;  s/project_name/$PROJECT_NAME/g ;  s/test:latest/$MS_NAME:$BUILD_TIMESTAMP/g" azure-vote-all-in-one-redis.yaml
            cat azure-vote-all-in-one-redis.yaml 
            kubectl  apply -f  azure-vote-all-in-one-redis.yaml  --kubeconfig $KUBETEST_CREDS  --namespace $PROJECT_NAME-$ENVIRONMENT
            """)
         }
      }

// echo ""
// echo "waiting for hello pod to be ready..."
// while [[ $(kubectl get pods -l app=hello -o 'jsonpath={..status.conditions[?(@.type=="Ready")].status}') != "True" ]]; do echo "waiting for pod" && sleep 1; done
// echo "pod is ready"      

      stage('Check App Started') {
         when {
            branch 'master'
         }
         environment {
            ENVIRONMENT = 'prod'
         }
         steps {
            echo "Check App Successfull deployed  to ${ENVIRONMENT}"
                timeout(5) {
            		waitUntil {
                        script {
                            def r = sh(script:" if [ \"\$(kubectl get pod --kubeconfig $KUBETEST_CREDS -o jsonpath=\"{..ready}\" -l app=$MS_NAME --namespace $PROJECT_NAME-$ENVIRONMENT)\" = \"false\" ]; then exit 1; else exit 0; fi", returnStatus: true)
                            return (r == 0);
                        }
          	  		}
       	 		}
         }
      }
}
    post {
        always {
            cleanWs()
        }
    }
}
