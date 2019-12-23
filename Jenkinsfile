pipeline {
    agent any
    stages{
      stage('PreCommit job') {
          git branch: 'dev', url: 'https://github.com/avkdr/spring-petclinic.git'
          sh 'mvn clean install'
          archiveArtifacts 'target/*.jar'
      }
    }
}
