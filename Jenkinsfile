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
        SONAR_SCANNER = tool "sonar-scanner"
        SONAR_SERVER = "sonar-server"
        NEXUS_LOGIN = "nexus-login"
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin"
        NEXUS_IP = "172.31.42.81"
        NEXUS_PORT = "8081"
        NEXUS_GRP_REPO = "group"
        NEXUS_RELEASE = "release"
        NEXUS_CENTRAL = "central"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: "ci-jenkins", credentialsId: "github-ssh", url: "git@github.com:Ujwal-s-Projects/vprofile-project.git" 
            }
        }

        stage("Build") {
            steps {
                sh "mvn -s settings.xml install -DskipTests"
            }
            post {
                success {
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        stage("Unit_Test") {
            steps {
                sh "mvn -s settings.xml test"
            }
        }

        stage("Checkstyle_Test") {
            steps {
                sh "mvn -s settings.xml checkstyle:checkstyle"
            }
        }

        stage("Code_Analysis") {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh ''' ${SONAR_SCANNER}/bin/sonar-scanner \
                    -Dsonar.projectKey=vpro-key \
                    -Dsonar.projectName=vpro-project \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.checkstyle.reportsPath=target/checkstyle-results.xml \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ '''
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
