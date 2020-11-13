pipeline {
    agent any
    stages {
        stage ('WS1') {
            when {
                expression { params.WS1 }
            }
            steps {
                echo "Hello, WS3! ou WS2"
            }
        }
        stage ('WS2') {
            when {
                expression { params.WS2 }
            }
            steps {
                echo "Hello, WS2"
            }
        }
        stage ('WS3') {
            when {
                expression { params.WS3 }
            }
            steps {
                echo "Hello, WS3! "
            }
        }
    }
}
