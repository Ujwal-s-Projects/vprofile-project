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

        stage("Code-Analysis") {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}
