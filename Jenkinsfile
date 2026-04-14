pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build + Tests + Coverage') {
            steps {
                bat 'mvn clean verify -B'
            }
            post {
                always {
                    junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
                    junit testResults: '**/target/failsafe-reports/*.xml', allowEmptyResults: true
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
                }
            }
        }

        stage('Qualite statique') {
            steps {
                bat 'mvn checkstyle:checkstyle pmd:pmd pmd:cpd spotbugs:spotbugs -B'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/checkstyle-result.xml, target/pmd.xml, target/cpd.xml, target/spotbugsXml.xml', allowEmptyArchive: true
                }
            }
        }
    }

    post {
        failure {
            emailext(
                subject: "Build FAILED - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Le build a échoué.

Projet : ${env.JOB_NAME}
Build : #${env.BUILD_NUMBER}
URL : ${env.BUILD_URL}
""",
                to: "ismaelcherif2023@gmail.com"
            )
        }
        success {
            echo "Build réussi !"
        }
    }
}