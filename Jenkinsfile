pipeline {
    agent any

    tools {
        maven "Maven 3.6.2"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "35.210.215.21:8081"
        NEXUS_REPOSITORY = "petclinic-snapshots"
        NEXUS_CREDENTIAL_ID = "nexus"
        MAVEN_PROJECT_VERSION= sh (
          script: "mvn -q -Dexec.executable=echo -Dexec.args='${project.version}' exec:exec |sed 's/[a-zA-Z<>/-]//g;s/[.]*$//'",
          returnStdout: true
          ).trim()
        TIMESTAMP= sh (
          script: 'date "+%Y%m%d.%H%M%S"',
          returnStdout: true
          ).trim()
        GIT_HASH=$(git log -1 --pretty=%h)
        MAVEN_UPDATED_PROJECT_VERSION="${MAVEN_PROJECT_VERSION}-${TIMESTAMP}-${GIT_HASH}"
    }

    stages {
      stage('clone code') {
          steps {
            git branch: 'dev', url: "https://github.com/avkdr/spring-petclinic.git"
          }
      }

      stage('mvn build') {
          steps {
            sh 'mvn clean package'
          }
      }

      stage('upload artifact to nexus') {
        steps {
          sh 'curl -v -u ${NEXUS_CREDENTIAL_ID} --upload-file ./target/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar http://35.210.215.21:8081/repository/petclinic-snapshots/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar \
          echo bla bla \n'
          }
        }
      }
    }
