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
          script {
          MAVEN_PROJECT_VERSION=$(mvn -q -Dexec.executable=echo -Dexec.args='${project.version}' --non-recursive exec:exec |sed 's/[a-zA-Z<>\/-]//g;s/[.]*$//')
          TIMESTAMP=$(date "+%Y%m%d.%H%M%S")
          GIT_HASH=$(git log -1 --pretty=%h)
          MAVEN_UPDATED_PROJECT_VERSION="${MAVEN_PROJECT_VERSION}-${TIMESTAMP}-${GIT_HASH}"
          echo "VERSION=${MAVEN_UPDATED_PROJECT_VERSION}" > env.properties
          }
        }
      }
    }
}
