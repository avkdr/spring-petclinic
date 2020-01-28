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
              sh "mvn versions:set -DnewVersion=${MAVEN_UPDATED_PROJECT_VERSION}"
              sh "mvn clean package -DskipTests"
            }
          }
      }

      stage('upload artifact to nexus') {
        steps {
          script {
              withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
              MAVEN_UPDATED_PROJECT_VERSION = MAVEN_UPDATED_PROJECT_VERSION.trim()
              echo MAVEN_UPDATED_PROJECT_VERSION
//               sh script: 'echo "curl -v -u ${USER}:${PASS} --upload-file ./target/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar http://35.210.215.21:8081/repository/petclinic-snapshots/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar"'
              sh script: "curl -v -u ${USER}:${PASS} --upload-file './target/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar' http://35.210.215.21:8081/repository/petclinic-snapshots/spring-petclinic-${MAVEN_UPDATED_PROJECT_VERSION}.jar"
                }
              }
            }
          }

      stage('release') {
        steps {
          script {
            withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
            BRANCH_NAME = sh script: "release-\$(mvn -q -Dexec.executable=echo -Dexec.args='\${MAVEN_UPDATED_PROJECT_VERSION}' --non-recursive exec:exec | sed 's/[a-zA-Z<>/-]//g;s/[.]*$//' | head -c 3)", returnStdout: true
            DEVELOPMENT_VERSION = \$(mvn -q -Dexec.executable=echo -Dexec.args='\${MAVEN_UPDATED_PROJECT_VERSION}' --non-recursive exec:exec | awk -F. -v OFS=. '{$2+=1} {print$0}'), returnStdout: true
            GIT_USER = 'avkdr'
            GIT_USER_MAIL = 'radkova.irina.a@gmial.com'
            CONNECTION_STRING = 'git@github.com:avkdr/spring-petclinic.git'
              sh "git config --global user.name '${GIT_USER}'"
              sh "git config --global user.email '${GIT_USER_MAIL}'"
              sh "git remote set-url origin ${CONNECTION_STRING}"
              sh "mvn release:clean release:branch -Dusername=${USERNAME} -Dpassword=${PASSWORD}"
            }
          }
        }
      }

        }
      }
