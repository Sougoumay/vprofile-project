pipeline {
    agent any

    tools {
        maven "MAVEN3.9",
        jdk "JDK17"
    }

    stages {
        stage('Fetch code') {
            steps {
                git branch: 'jenkins_CI',
                url: 'https://github.com/Sougoumay/vprofile-deploy-beanstalk'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Archiving artifact'
                    archiveArtifacts artifacts: '**/*.war'

                }
            }
        }
    }
}