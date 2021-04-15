pipeline {
   agent { label 'master'}
   stages {
      stage('Verify Branch') {
         steps {
            echo "$GIT_BRANCH"
         }
      }
   }
}
