pipeline {
    agent none
    stages {
        stage("worker-build") {
            agent {
                docker {
                   image 'maven:3.6.1-jdk-8-slim'
                   args '-v $HOME/.m2:/root/.m2'
                 }
            }
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
    
        stage("result-build") {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset "**/result/**"
            }
            steps {
                echo 'Compiling worker app'
                dir('result') {
                    sh 'npm install'
                }
            }
        }
        stage("result-test") {
            agent {
                docker {
                    image 'node:8.16.0-alpine'                
                }
            }
            steps {
                echo 'Running Unit Test on worker app'
                dir ('result') {
                    sh 'pwd'
                    sh 'npm install'
                    sh 'npm test'                
                }
            }
        }
        stage("result-package") {
            steps {
                echo 'Packaging worker app'
            }
        }
        
        stage ('vote-build') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset '**/vote/**'
            }
            steps {
                echo 'compilando vote app'
                dir(path: 'vote') {
                    sh 'pip install -r requeriments.txt'
                }
            }
        }
        stage ('vote-test') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset '**/vote/**'
            }
            steps {
                echo 'Corre las pruebas unitarias de vote app'
                dir(path: 'vote') {
                    sh 'pip install -r requeriments.txt'
                    sh 'nosetest -v'
                }
            }            
        } 
        stage ('vote-integration') {
            agent any
            when {
                changeset '**/vote/**'
                branch 'master'
            }
            steps {
                echo 'Se corren las pruebas de integracion de vote app'
                dir (path: 'vote') {
                    sh 'sh integration_test.sh'
                }
            }
        }
        
        stage ('vote-docker-package') {
            agent any
            when {
                changeset '**/vote/**'
                branch 'master'
            }
            steps {
                echo 'Empaquetando aplicacion vote app'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
                        def voteImage = docker.build("cafavila/vote:${env.GIT_COMMIT}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")
                        voteImage.push("latest")
                    }
                }
            }
        }

        stage ('Sonarqube') {
            agent any
            when {
                branch 'master'
            }
            environment {
                sonarpath = tool 'SonarScanner'
            }
            steps {
                echo 'Ejecutando Sonarqube Analisis....'
                withSonarQubeEnv('sonar-instavote') {
                    sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
            
        stage ('deploy to dev') {
            agent any
            when {
                branch 'master'
            }
            steps {
                echo 'Deployando instavote con docker compose'
                sh 'docker compose up -d'
            }
        }
    }

    post {
        always {
            echo 'Construccion de aplicacion instavote is completed..'
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
