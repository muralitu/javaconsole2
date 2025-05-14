pipeline {
    agent any

    stages {
        stage('SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/naresha/javaconsole2.git'
            }
        }
        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }

    }
}