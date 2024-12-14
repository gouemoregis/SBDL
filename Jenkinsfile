pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
               // Ensure Python and pip are available
                sh 'python3 --version || sudo apt-get update && sudo apt-get install -y python3 python3-pip'

                // Install pipenv
                sh 'pip install --user pipenv'

                // Remove old virtual environment if it exists
                sh 'pipenv --rm || true'

                // Sync dependencies
                sh 'pipenv --python python3 sync'
            }
        }
        stage('Test') {
            steps {
                sh 'pipenv run pytest'
            }
        }
        stage('Package') {
            when {
                anyOf { branch 'main'; branch 'release' }
            }
            steps {
                sh 'zip -r sbdl.zip lib'
            }
        }
        stage('Release') {
            when {
                branch 'release'
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-key-id', keyFileVariable: 'SSH_KEY')]) {
                    sh "scp -i $SSH_KEY -o 'StrictHostKeyChecking no' -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf ubuntu@54.242.173.177:/home/ubuntu/sbdl-qa"
                }
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'jenkins-key-id', keyFileVariable: 'SSH_KEY')]) {
                    sh "scp -i $SSH_KEY -o 'StrictHostKeyChecking no' -r sbdl.zip log4j.properties sbdl_main.py sbdl_submit.sh conf ubuntu@54.242.173.177:/home/ubuntu/sbdl-prod"
                }
            }
        }
    }
}
