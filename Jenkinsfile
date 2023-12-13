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
        SONARSERVER = "sonarserver"
        SONARSCANNER = "sonarscanner"
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

        stage("Sonar_Analysis") {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
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
    }
}