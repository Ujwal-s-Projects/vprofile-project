pipeline {
    agent any 

/*
    tools {
        maven "MAVEN3"
        sonarqube "sonarscanner"
    }
*/
    stages {
        
        stage("Build") {
            steps {
                sh 'mvn clean install -DskipTests'
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
                sh 'mvn test'
            }
        }

        stage("Integration_Test") {
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage("Code_Analysis_With_Checkstyle") {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
    }
}
