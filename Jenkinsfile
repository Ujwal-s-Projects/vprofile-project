def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any

    tools {
        jdk 'jdk11'
        maven 'MAVEN3'
    }

    environment {
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        NEXUS_LOGIN = 'nexus-login'
        NEXUSIP = '172.31.10.238'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'group-repo'
        RELEASE_REPO = 'release-repo'
        SNAP_REPO = 'snap-repo'
        CENTRAL_REPO = 'central-repo'
        SONARSCANNER = 'sonar-scanner'
        SONARSERVER = 'sonar-server'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'ci-jenkins', credentialsId: 'github-ssh-connection', url: 'git@github.com:Ujwal-s-Projects/vprofile-project.git'
                echo 'success.......................'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -s settings.xml clean install -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('Unit-Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Code-Analysis-Checkstyle') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('SonarQube-Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                    withSonarQubeEnv("${SONARSERVER}") {
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
        stage("Quality_Gates") {
            steps {
                timeout(time: 10, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Nexus_Artifact") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'My-App',
                    version: "${env.BUILD_ID}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'My-App',
                         classifier: '',
                         file: 'target/vprofile-v2.war',
                         type: 'war']
                    ]
                )

            }
        }
    }

    post {
        always {
               script {
            echo 'slack notification'
            slackSend channel: '#jenkins-ci_cd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n more info at: ${env.BUILD_URL}"
        }
     }
    }
}
