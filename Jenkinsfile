def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE' : 'danger'
]

pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    stages {
        stage('Fetch code') {
            steps {
                git branch: 'jenkins_CI',
                url: 'https://github.com/Sougoumay/vprofile-project'
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

         stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
         }

         stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
         }

         stage('Sonar code Analysis') {
            environment {
                scannerHome = tool 'Sonar6.2'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
         }

         stage("Quality Gate") {
             steps {
               timeout(time: 1, unit: 'HOURS') {
                 waitForQualityGate abortPipeline: true
               }
             }
         }

         stage('Upload Artifact') {
             steps {
                 nexusArtifactUploader(
                     nexusVersion: 'nexus3',
                     protocol: 'http',
                     nexusUrl: '172.31.91.190:8081',
                     groupId: 'QA',
                     version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                     repository: 'vprofile-repo',
                     credentialsId: 'nexuslogin',
                     artifacts: [
                         [artifactId: 'vproapp',
                          classifier: '',
                          file: 'target/vprofile-v2.war',
                          type: 'war'
                         ]
                     ]
                 )
             }
         }
    }

    post {
        always {
            echo 'Slack Notification'
            slackSend channel: '#devopscicd',
            color : COLOR_MAP[currentBuild.currentResult],
            message: "Find Status of Pipeline:- ${currentBuild.currentResult}:* Job ${env.JOB_NAME} ${env.BUILD_NUMBER} \n For more information go to url ${BUILD_URL}"
        }
    }
}