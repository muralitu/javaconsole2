pipeline {
    agent any

    environment {
        VERSION = "${env.BUILD_NUMBER}"
        JAVA_HOME = tool name: 'jdk', type: 'jdk'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build Project') {
            steps {
                script {
                    try {
                        sh "./gradlew clean build -PbuildVersion=${VERSION} test"
                        echo "✅ Build completed successfully."
                    } catch (err) {
                        echo "❌ Build failed: ${err}"
                        throw err
                    } finally {
                        echo "ℹ️ Build stage completed."
                    }
                }
            }
        }

        stage('Static Analysis') {
            steps {
                sh './gradlew pmdMain'
            }
        }

        stage('Code Convention Check') {
            steps {
                sh './gradlew checkstyleMain'
            }
        }

        stage('Run Tests & Coverage') {
            steps {
                sh './gradlew test jacocoTestReport'
            }
        }

        stage('Publish Reports') {
            steps {
                junit 'app/build/test-results/**/*.xml'
                jacoco(
                    execPattern: 'app/build/jacoco/*.exec',
                    classPattern: 'app/build/classes',
                    sourcePattern: 'src/main/java'
                )
                recordIssues(
                    tools: [
                        checkStyle(pattern: 'app/build/reports/checkstyle/*.xml'),
                        pmdParser(pattern: 'app/build/reports/pmd/*.xml')
                    ]
                )
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'app/build/libs/*.war', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "🎉 Build #${env.BUILD_NUMBER} succeeded!"
        }
        failure {
            echo "🚨 Build #${env.BUILD_NUMBER} failed!"
        }
        always {
            echo "📦 Build #${env.BUILD_NUMBER} completed."
        }
    }
}
