@Library('github.com/verticaio/demo-shared-pipeline.git') _ 

library identifier: 'github.com/verticaio/demo-shared-pipeline.git',
    //'master' refers to a valid git-ref
    //'mylibraryname' can be any name you like
    retriever: modernSCM([
      $class: 'GitSCMSource',
      credentialsId: 'GitHubConnection',
      remote: 'https://github.com/verticaio/demo-shared-pipeline.git'
])

pipeline{
    agent any
    stages{
        stage("Call Library Function"){
            steps{
                script { helloWorld() }
            }
        }
    }
}