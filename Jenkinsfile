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
        pom = readMavenPom file: "pom.xml"
        version = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true
        timeStamp = Calendar.getInstance().getTime().format('YYYYMMdd-hhmmss',TimeZone.getTimeZone('CST'))
        MAVEN_PROJECT_VERSION = ''
        GIT_HASH = ''
        MAVEN_UPDATED_PROJECT_VERSION = ''
    }

    stages {
      stage('clone code') {
          steps {
            git branch: 'dev', url: "https://github.com/avkdr/spring-petclinic.git"
          }
      }

      stage('make version') {
        steps {
          script {
          MAVEN_PROJECT_VERSION = sh script: 'mvn -q -Dexec.executable=echo -Dexec.args="${version}" exec:exec |sed "s/[a-zA-Z<>/-]//g;s/[.]*$//"', returnStdout: true
          MAVEN_PROJECT_VERSION = MAVEN_PROJECT_VERSION.trim()
          GIT_HASH = sh script: 'git log -1 --pretty=%h', returnStdout: true
          MAVEN_UPDATED_PROJECT_VERSION = "${MAVEN_PROJECT_VERSION}-${timeStamp}-${GIT_HASH}"
          println MAVEN_UPDATED_PROJECT_VERSION
          }
        }
      }

      stage('mvn build') {
          steps {
            script {
              sh script: 'mvn clean package -Dbuild.number=${MAVEN_UPDATED_PROJECT_VERSION}', returnStdout: true
            }
          }
      }

      stage('upload artifact to nexus') {
        steps {
          sh 'curl -v -u ${NEXUS_CREDENTIAL_ID} --upload-file ./target/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar http://35.210.215.21:8081/repository/petclinic-snapshots/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar'
          }
        }
      }
    }
