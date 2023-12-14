pipeline {
    agent any 
   
    tools {
        jdk "JDK11"
        maven "MAVEN3"
    }

    stages {
        stage("Checkout") {
            steps {
                git branch: "ci-jenkins", url: "git@github.com:Ujwal-s-Projects/vprofile-project.git"
            }
        }
    }
}
