@Library('github.com/verticaio/demo-shared-pipeline.git') _ 

pipeline{
    agent any
    stages{
        stage("Call Library Function with an Argument"){
            steps{
                script { helloWorld() }
            }
        }
    }
}