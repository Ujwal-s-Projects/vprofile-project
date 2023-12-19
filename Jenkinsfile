def COLOR_MAP = [
    "SUCCESS": "good",
    "FAILURE": "danger",
]
pipeline {
    agent any 

    tools {
        jdk "jdk11"
        maven "Maven3"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: "ci-jenkins", credentialsId: "github-ssh", url: "git@github.com:Ujwal-s-Projects/vprofile-project.git" 
            }
        }

        stage("Build") {
            steps {
                sh "mvn install -DskipTests"
            }
            post {
                success {
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        stage("Unit_Test") {
            steps {
                sh "mvn test"
            }
        }

        stage("Checkstyle_Test") {
            steps {
                sh "mvn checkstyle:checkstyle"
            }
        }

        stage("Slack_Notification") {
            steps {
                slackSend channel: '#jenkins',
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "${currentBuild.currentResult}: ${env.JOB_NAME} - ${env.BUILD_NUMBER} - ${env.BUILD_URL}"
            }
        }
    }
}