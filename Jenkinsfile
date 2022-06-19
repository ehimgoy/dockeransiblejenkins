pipeline{
    agent any
    tools {
        maven 'maven3'
    }
    environment{
        DOCKER_TAG = getVersion()
    }
    stages{
        stage('SCM'){
            steps{
                git credentialsId: 'gitpassnew', url: 'https://github.com/ehimgoy/dockeransiblejenkins.git'
            }
        }
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }   
        }
        stage('Docker Build'){
            steps{
                sh "docker build . -t ehimgoy/jenkinspipe:${DOCKER_TAG}"
            }
        }
        stage('Dockerhub push'){
            steps{
                withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhubpwd')]) {
                    sh 'docker login -u ehimgoy -p ${dockerhubpwd}'
                }
                sh "docker push ehimgoy/jenkinspipe:${DOCKER_TAG}"
            }
        }
        stage('Docker deploy'){
            steps{
                ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'Ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'    
            }
        }
        
    }
}

def getVersion(){
    def commitHash = sh returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
