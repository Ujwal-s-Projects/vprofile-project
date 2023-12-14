pipeline {
    tools {
        maven "MAVEN3"
    }

    stages {
        stage("Checkout") {
            git branch: "ci-jenkins" url: "git@github.com:Ujwal-s-Projects/vprofile-project.git"
        }
    }
}