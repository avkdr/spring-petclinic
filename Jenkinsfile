pipeline {
    agent any
    stages {
    stage ('PreCommit job') {
      steps {
      git branch: 'dev', url: 'https://github.com/avkdr/spring-petclinic.git'
      sh 'mvn clean package'
      archiveArtifacts 'target/*.jar'
      }
    }
  }
}
