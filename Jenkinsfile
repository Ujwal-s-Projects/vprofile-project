pipeline {
    agent any 

/*
    tools {
        maven "MAVEN3"
        sonarqube "sonarscanner"
    }
*/
    environment {
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vpro-maven-central"
        SNAP_REPO = "vprofile-snapshop" 
        NEXUSIP = "172.31.0.147" 
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "vpro-maven-group"
        NEXUS_LOGIN = "nexuslogin" 
    }
    stages {

        stage("Build") {
            steps {
                sh 'mvn -s settings.xml install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage("Unit_Test") {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage("Code_Analysis_With_Checkstyle") {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
    }
}