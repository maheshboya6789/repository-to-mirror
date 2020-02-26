pipeline {
    agent {label 'mynode1'}
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('git clone')
        {
        steps{
            sh'rm -rf node-app'
            sh' git clone https://github.com/anilkumarpuli/node-app.git'
        }
        }
        stage('build'){
            steps{
                sh'mvn clean install'
            }
        }
          
        stage('nexus upload')
        {
            steps{
               nexusArtifactUploader artifacts: [[artifactId: 'hai', classifier: 'java', file: 'target/vprofile-v1.war', type: 'war']], credentialsId: 'nexu-id', groupId: 'mygroup', nexusUrl: 'ec2-13-59-36-246.us-east-2.compute.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'mavenrepo', version: '$BUILD_ID'
                 }
        }
        stage('deploy to tomcat')
        {
            steps
            {
  deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://ec2-3-19-57-8.us-east-2.compute.amazonaws.com:8080')], contextPath: 'anil', war: 'target/vprofile-v1.war'        }
    }
        stage('Build Docker Image'){
            steps{

               sh "docker build . -t anilkumblepuli/java2:${DOCKER_TAG}"
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')])  {
                    
                    sh "docker login -u anilkumblepuli -p ${dockerhub}"
                    sh "docker push anilkumblepuli/java2:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['kops-machine']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ubuntu@172.31.14.30:/home/ubuntu/"
                    script{
                        try{
                            
                            sh "ssh ubuntu@172.31.14.30 kubectl apply -f ."
                        }catch(error){
                            sh "ssh ubuntu@172.31.14.30 kubectl create -f ."
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
