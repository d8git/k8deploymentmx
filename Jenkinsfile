pipeline {
    agent any

    triggers {
        githubPush()
    }

    stages {
        stage('Test') {
            steps {
                echo 'Triggered'
            }
        }
    }
}
