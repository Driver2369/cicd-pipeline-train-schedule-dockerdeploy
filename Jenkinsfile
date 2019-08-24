pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image'){
            when{
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("driver23/train-schedule")
                    // saves the results of the docker.build call and then it can be called inside the inside block and it will perform smoke test if it is running 
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login'){
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
   //             withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        sh "sshpass -p 'jenkins' -v ssh -o StrictHostKeyChecking=no deploy@3.88.59.214 \"docker pull driver23/train-schedule:latest\""
    //                    try {
    //                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_id \"docker stop train-schedule\""
    //                        sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_id  \"docker rm train-schedule\""
    //                    } catch (err) {
    //                        echo: 'caught error: $err'
    //                    }
    //                    sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@prod_id \"docker run --restart always --name train-schedule -p 8080:8080 -d driver23/train-schedule:${env.BUILD_NUMBER}\""
   //                 }
                } 
            }
        }
    }
}
