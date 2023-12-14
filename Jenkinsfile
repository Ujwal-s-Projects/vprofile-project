pipeline {
    agent any 
   
    tools {
        jdk "JDK11"
        maven "MAVEN3"
    }

    environment {
        NEXUS_USER="admin"
        NEXUS_PASS="admin"
        NEXUS_LOGIN="nexuslogin"
        NEXUSIP="172.31.32.196"
        NEXUSPORT="8081"
        NEXUS_GRP_REPO="my-group"
        CENTRAL_REPO="my-central"
        RELEASE_REPO="my-release"
        SONARSCANNER="sonar-scanner"
        SONARSERVER="sonar-server"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: 'ci-jenkins', credentialsId: 'github-ssh-key', url: 'git@github.com:Ujwal-s-Projects/vprofile-project.git'
                echo "success......................."
            }
        }

        stage("Build") {
            steps {
                sh 'mvn -s settings.xml install -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage("Unit-Test") {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage("Code-Analysis-Checkstyle") {
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
                timeout(10) {
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
                    repository: "${NEXUS_RELEASE}",
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
}
