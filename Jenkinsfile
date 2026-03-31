pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
            }
        }

        stage('Test') {
            steps {
                sh 'cat index.html'
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'index.html', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "Success for ${env.BRANCH_NAME}"
        }
        failure {
            echo "Failed"
        }
    }
}
