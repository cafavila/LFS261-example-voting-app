pipeline {
    agent {
        docker {
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:root/.m2'
        }
    }
    stages {
        stage("worker-build") {
            when {
                changeset "**/worker/**"
            }            
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile'
                }
            }
        }
        stage("worker-test") {
            when {
                changeset "**/worker/**"
            }
            steps {
                echo 'Running Unit Test on worker app'
                dir('worker') {
                    sh 'mvn clean test'
                }
            }
        }
        stage("worker-package") {
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker') {
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('worker-docker-package') {
            agent any
            when {
                changeset "**/worker/**"
                branch 'master'
            }
            steps {
                echo 'packaging worker app with docker'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def workerImage = docker.build("cafavila/worker:v${env.BUILD_ID}", "./ worker")
                        workerImage.push()
                        workerImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Building multibranch pipeline for worker is completed..'
        }
        failure {
            echo 'Algo fallo, revisar'
     //       slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }
        success {
            echo 'Exitoso!!'
     //       slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
        }        
    }
}
