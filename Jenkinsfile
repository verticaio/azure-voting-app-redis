@Library('shared_lib@master') _ 


library identifier: 'shared_lib@master',
    //'master' refers to a valid git-ref
    //'mylibraryname' can be any name you like
    retriever: modernSCM([
      $class: 'GitSCMSource',
      credentialsId: 'GitHubConnection',
      remote: 'https://github.com/verticaio/demo-shared-pipeline.git'
])

echoPipeline() {
    message = 'Salam Test'
    message2 = 'Salam1'
}