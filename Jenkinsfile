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
                  // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                  pom = readMavenPom file: "pom.xml";
                  // Find built artifact under target folder
                  filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                  // Print some info from the artifact found
                  echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                  // Extract the path from the File found
                  artifactPath = filesByGlob[0].path;
                  // Assign to a boolean response verifying If the artifact name exists
                  artifactExists = fileExists artifactPath;

                  MAVEN_PROJECT_VERSION=pom.artifactId;
                  TIMESTAMP=java.time.LocalDateTime.now();
                  GIT_HASH=checkout(scm).GIT_COMMIT;
                  MAVEN_UPDATED_PROJECT_VERSION="MAVEN_PROJECT_VERSION - TIMESTAMP - GIT_HASH";
                  def name = MAVEN_UPDATED_PROJECT_VERSION;
                  echo "name"

                  if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
          }
        }
      }
    }
}
