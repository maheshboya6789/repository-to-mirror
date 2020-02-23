pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    tools {
      // Install the Maven version configured as "M3" and add it to the path.
      maven "maven"
   }

   stages {
      stage('Build') {
         steps {
            git 'https://github.com/jglick/simple-maven-project-with-tests.git'
            sh "mvn -Dmaven.test.failure.ignore=true clean package"

           }

         post {
            success {
               junit '**/target/surefire-reports/TEST-*.xml'
               archiveArtifacts 'target/*.war'
            }
         }
      }
   }

        stage('nexus upload')
        {
            steps{nexusArtifactUploader artifacts: [[artifactId: 'java', classifier: 'artifact', file: 'tartget/vprofile-v1.war', type: 'war']], credentialsId: 'nexu-id', groupId: 'vprofile', nexusUrl: '172.31.19.30:8081/nexus', nexusVersion: 'nexus2', protocol: 'http', repository: 'releases', version: '$BUILD_ID'
                 }
        }
        stage('deploy to tomcat')
        {
            steps
            {
      deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://172.31.16.25:8080')], contextPath: 'artifact', war: 'target/vprofile-v1.war'            
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
