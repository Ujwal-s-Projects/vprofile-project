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
    }
}