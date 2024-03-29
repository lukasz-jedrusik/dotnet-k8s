pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image') {
            steps{
                sh "docker build . -t kwazi1984/dotnet-k8s:${DOCKER_TAG} "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u kwazi1984 -p ${dockerHubPwd}"
                    sh "docker push kwazi1984/dotnet-k8s:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to Kubernetes'){
            steps{
                sh "chmod +x k8s-test/changeTag.sh"
                sh "./k8s-test/changeTag.sh ${DOCKER_TAG}"
                sshagent(['kops-machine']) {
                    //sh "ssh -o StrictHostKeyChecking=no ubuntu@10.0.2.15 kubectl version"
                    sh "scp -o StrictHostKeyChecking=no k8s-test/k8s-deploy.yaml k8s-test/k8s-svc-clusterip.yaml k8s-test/k8s-ingress.yaml ubuntu@10.0.2.15:/home/ubuntu/jenkinks-k8s-ssh/dotnet-test/"
                    script{
                        try{
                            sh "ssh ubuntu@10.0.2.15 kubectl apply -f /home/ubuntu/jenkinks-k8s-ssh/dotnet-test/. -n dotnet-test"
                        }catch(error){
                            sh "ssh ubuntu@10.0.2.15 kubectl create -f /home/ubuntu/jenkinks-k8s-ssh/dotnet-test/. n dotnet-test"
                        }
                    }
                }        
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}