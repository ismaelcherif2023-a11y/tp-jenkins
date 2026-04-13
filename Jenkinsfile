/**
 * Jenkinsfile – Pipeline CI complète (Windows + Linux compatible)
 * Projet : Boutique en ligne – ICDE848
 */

pipeline {

    agent any

    tools {
        maven 'Maven3'
        jdk   'JDK17'
    }

    parameters {
        string(
            name: 'BRANCH',
            defaultValue: 'main',
            description: 'Branche Git à builder'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Environnement de déploiement'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Ignorer les tests'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch : ${env.GIT_BRANCH}"
                echo "Commit : ${env.GIT_COMMIT}"
            }
        }

        stage('Build') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn clean compile -B'
                    } else {
                        bat 'mvn clean compile -B'
                    }
                }
            }
        }

        stage('Tests unitaires') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn test -B'
                    } else {
                        bat 'mvn test -B'
                    }
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Tests intégration') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn verify -Dsurefire.skip=true -B'
                    } else {
                        bat 'mvn verify -Dsurefire.skip=true -B'
                    }
                }
            }
            post {
                always {
                    junit '**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Couverture JaCoCo') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn jacoco:report -B'
                    } else {
                        bat 'mvn jacoco:report -B'
                    }
                }
            }
            post {
                always {
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java',
                        minimumLineCoverage: '70'
                    )
                }
            }
        }

        stage('Qualité') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            mvn checkstyle:checkstyle \
                                pmd:pmd \
                                pmd:cpd \
                                spotbugs:spotbugs \
                                -B
                        '''
                    } else {
                        bat '''
                            mvn checkstyle:checkstyle ^
                                pmd:pmd ^
                                pmd:cpd ^
                                spotbugs:spotbugs ^
                                -B
                        '''
                    }
                }
            }
            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            checkStyle(pattern: '**/checkstyle-result.xml'),
                            pmdParser(pattern: '**/pmd.xml'),
                            cpd(pattern: '**/cpd.xml'),
                            spotBugs(pattern: '**/spotbugsXml.xml')
                        ]
                    )
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: '**/target/*.jar',
                    fingerprint: true
                )
                echo "Artefact archivé"
            }
        }
    }

    post {

        always {
            echo "Pipeline terminée — statut : ${currentBuild.currentResult}"
        }

        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build échoué

Job   : ${env.JOB_NAME}
Build : ${env.BUILD_NUMBER}
URL   : ${env.BUILD_URL}
                """,
                to: 'ismaelcherif2023@gmail.com'
            )
        }

        fixed {
            emailext(
                subject: "✅ FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build redevenu stable : ${env.BUILD_URL}",
                to: 'ismaelcherif2023@gmail.com'
            )
        }
    }
}