pipeline {
    agent { label 'rahul-b12' }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SHArahul/TravelMemory.git'
            }
        }
        stage('Install') {
            steps {
                sh 'cd backend; npm install'
            }
        }
        stage('Build') {
            steps {
                sh 'cd backend; npm run build'
            }
        }
    }

}




