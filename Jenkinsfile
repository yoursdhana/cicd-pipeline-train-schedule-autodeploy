pipeline {
    tools{
        jdk 'java11'
    }    
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub usernamee
        DOCKER_IMAGE_NAME = "yoursdhana/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            //when {
                //branch 'master'
            //}
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            //when {
                //branch 'master'
            //}
            steps {
                script {
                    withDockerRegistry([ credentialsId: "dockerHub", url: "" ]) {
                        app.push()
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            //when {
                //branch 'master'
            //}
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                sh ("kubectl --kubeconfig  /home/centos/.kube/config apply -f train-schedule-kube-canary.yml")
            }
        }
        stage('DeployToProduction') {
            //when {
                //branch 'master'
            //}
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh ("kubectl apply -f train-schedule-kube-canary.yml")
                sh ("kubectl apply -f train-schedule-kube.yml")
            }
        }
    }
}
