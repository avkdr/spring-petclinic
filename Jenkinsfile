pipeline {
    agent any
    stage('PreCommit job') {
        git branch: 'dev', url: 'https://github.com/avkdr/spring-petclinic.git'
        sh 'mvn clean package'
        archiveArtifacts 'target/*.jar'
    }
}