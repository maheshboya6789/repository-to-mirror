pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
       stage('code cloning')
        {
         steps{
           sh'git clone 'https://github.com/anilkumarpuli/node-app.git'
               }
           }
         stage('code build by maven'){
               steps{
               sh'mvn install'
           }
          } 
        stge('nexus upload')
        {
            steps{nexusArtifactUploader artifacts: [[artifactId: 'java', classifier: 'artifact', file: 'tartget/vprofile-v1.war', type: 'war']], credentialsId: 'nexu-id', groupId: 'vprofile', nexusUrl: '18.189.31.87:8081/nexus', nexusVersion: 'nexus2', protocol: 'http', repository: 'releases', version: '$BUILD_ID'
                 }
        }
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t anilkumblepuli/vprofile1:${DOCKER_TAG}"
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'dockerpsD', variable: 'dockerpsD')])  {
                    sh "docker login -u anilkumblepuli -p ${dockerpsD}"
                    sh "docker push anilkumblepuli/vprofile1:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['k8s-machine']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ubuntu@52.66.186.30:/home/ubuntu/"
                    script{
                        try{
                            sh "ssh ubuntu@52.66.186.30 kubectl apply -f ."
                        }catch(error){
                            sh "ubuntu@52.66.186.30 kubectl create -f ."
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
