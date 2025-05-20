pipeline {
    agent any
    tools {
        maven "MAVEN"
    }
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "0.0.0.0:8081"
        NEXUS_REPOSITORY = "repo"
        NEXUS_CREDENTIAL_ID = "Nexus_ID"
    }
    stages {
        stage("Clone code from GitHub") {
            steps {
                script {
                    git branch: 'main', url: 'https://github.com/Salhianis1/nexus-jenkins.git'
                }
            }
        }
        stage("Maven Build") {
            steps {
                script {
                    sh "mvn clean package -DskipTests=true"
                }
            }
        }
        stage("Publish to Nexus Repository Manager") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

                    if (filesByGlob.length == 0) {
                        error "No artifact found in target directory."
                    }

                    def artifactPath = filesByGlob[0].path
                    echo "Uploading artifact: ${artifactPath}, groupId: ${pom.groupId}, artifactId: ${pom.artifactId}, version: ${pom.version}"

                    if (fileExists(artifactPath)) {
                        nexusArtifactUploader(
                            nexusVersion: 'nexus3',
                            protocol: 'http',
                            nexusUrl: 'http://0.0.0.0:8081',
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: repo,
                            credentialsId: Nexus_ID,
                            artifacts: [
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: artifactPath,
                                    type: pom.packaging
                                ],
                                [
                                    artifactId: pom.artifactId,
                                    classifier: '',
                                    file: "pom.xml",
                                    type: "pom"
                                ]
                            ]
                        )
                    } else {
                        error "Artifact file does not exist: ${artifactPath}"
                    }
                }
            }
        }
    }
}
