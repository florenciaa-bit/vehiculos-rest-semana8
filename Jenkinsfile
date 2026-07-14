pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/florenciaa-bit/vehiculos-rest-semana8.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t vehiculos-rest .'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker stop vehiculos-rest || true'
                sh 'docker rm vehiculos-rest || true'
                sh 'docker run -d --name vehiculos-rest -p 9090:8080 vehiculos-rest'
            }
        }
    }
}
