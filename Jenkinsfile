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
         stage ('Invoke_pipeline2') {
            steps {
                build job: 'pipeline2',propagate: true, wait: true, parameters: [
                string(name: 'WS3', value: "test_param")
                ]
            }
        }
    }
	 post{ 
        always{  
            deleteDir() 
        }
}
}
