def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any 
   
    tools {
        jdk "jdk11"
        maven "Maven3"
    }

    environment {
        NEXUS_USER="admin"
        NEXUS_PASS="admin"
        NEXUS_LOGIN="nexus-login"
        NEXUS_IP="172.31.42.81"
        NEXUS_PORT="8081"
        NEXUS_GRP_REPO="group"
        NEXUS_CENTRAL="central"
        NEXUS_RELEASE="release"
        NEXUS_SNAP="snap"
        SONAR_SCANNER="sonar-scanner"
        SONAR_SERVER="sonar-server"
        REGISTRY_CREDS = "ecr:ap-south-1:aws_creds"
        REGISTRY_REPO = "814495875142.dkr.ecr.ap-south-1.amazonaws.com/my-repo"
        REGISTRY_URL = "https://814495875142.dkr.ecr.ap-south-1.amazonaws.com"
        CLUSTER = "staging-cluster"
        SERVICE = "staging-svc"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: 'cicd-jenkins', credentialsId: "github-ssh", url: 'git@github.com:Ujwal-s-Projects/vprofile-project.git'
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

        stage("Unit_Test") {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage("Code_Analysis_Checkstyle") {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('SonarQube_Analysis') {
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
                    nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",
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

        stage("Build_Docker_Img") {
            steps {
                script {
                    dockerImage = docker.build( REGISTRY_REPO + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
            }
        }

        stage("Upload_Img_To_ECR") {
            steps {
                script {
                    docker.withRegistry(REGISTRY_URL, REGISTRY_CREDS) {
                        dockerImage.push("$BUILD_NUMBER")
                    }
                }
            }
        }
        
        stage ("Deploy_To_Stage_ECS") {
            steps {
                withAWS(credentials: "aws_creds", region: "ap-south-1") {
                    sh 'aws ecs update-service --cluster ${CLUSTER} --service ${SERVICE} --force-new-deployment'
                }
            }
        }
    }

    post {
        always {
            echo 'slack notification'
            slackSend channel: '#jenkins',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n more info at: ${env.BUILD_URL}"
        }
    }
}
