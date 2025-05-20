pipeline {
    agent any

    tools {
        maven 'Maven 3.8.1' // Set this to your Maven installation in Jenkins
        jdk 'JDK 11'         // Set this to your configured JDK version
    }

    environment {
        // These are Jenkins credentials IDs
        OSSRH_USERNAME = credentials('sonatypeUsername')
        OSSRH_PASSWORD = credentials('sonatypePassword')
        GPG_PASSPHRASE = credentials('gpg.passphrase')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('Deploy to OSSRH') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    mvn clean deploy -Psign-artifacts,ossrh-deploy \\
                        -Dgpg.passphrase=$GPG_PASSPHRASE \\
                        -Dgpg.skip=false \\
                        -DskipTests=false
                """
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            junit 'target/surefire-reports/*.xml'
        }
        failure {
            mail to: 'dev-team@example.com',
                 subject: "Build failed: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                 body: "See Jenkins for details: ${env.BUILD_URL}"
        }
    }
}
