pipeline {
    agent none

    stages {
        stage('Checkout') {
            agent { label 'linux' }  // or 'windows' depending on your node
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/your-org/your-repo.git']]
                ])
            }
        }

        stage('Install Postman CLI') {
            agent { label 'linux' }
            steps {
                sh '''
                    if ! command -v postman &> /dev/null
                    then
                        echo "Installing Postman CLI..."
                        curl -o- "https://dl-cli.pstmn.io/install/linux64.sh" | sh
                    else
                        echo "Postman CLI already installed."
                    fi
                '''
            }
        }

        stage('Run API Tests') {
            parallel {
                stage('Linux Node') {
                    agent { label 'linux' }
                    steps {
                        sh 'postman collection run ./collections/my_collection.json -e ./environments/dev.json'
                    }
                }
                stage('Windows Node') {
                    agent { label 'windows' }
                    steps {
                        bat 'postman collection run .\\collections\\my_collection.json -e .\\environments\\dev.json'
                    }
                }
            }
        }

        stage('Publish Results') {
            agent { label 'linux' }
            steps {
                junit 'reports/*.xml'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished. Cleaning up...'
            cleanWs()
        }
    }
}
