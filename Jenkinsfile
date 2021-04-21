@Library('github.com/verticaio/demo-shared-pipeline.git') _ 

pipeline{
    agent any
    stages{
        stage("Call Library Hello Word Function"){
            steps{
                script { helloWorld() }
            }
            post{
                always{
                    echo "========always========"
                }
                success{
                    echo "========A executed successfully========"
                }
                failure{
                    echo "========A execution failed========"
                }
            }
        }
    }
    post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}