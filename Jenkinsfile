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

    environment {
        SONAR_SCANNER = "sonar-scanner"
        SONAR_SERVER = "sonar-server"
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

        stage("Code_Analysis") {
            environment {
                scannerHome = tool "${SONAR_SCANNER}"
            }
            steps {
                    withSonarQubeEnv("${SONAR_SERVER}") {
                        sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=my-key \
                        -Dsonar.projectName=my-project \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.checkstyle.reportsPath=target/checkstyle-result.xml \
                        -Dsonar.junit.reportsPath=target/surefire-reports/
                        '''
                    }
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