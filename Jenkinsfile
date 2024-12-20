def mainDir="./"
def ecrLoginHelper="docker-credential-ecr-login"
def region="ap-northeast-2"
def ecrUrl="615052872572.dkr.ecr.ap-northeast-2.amazonaws.com"
def repository="jk-test"
def deployAccount="jk"
def deployHost="172.17.0.176"
def jenkinsAwsKey="aws-key"

pipeline {
    agent any

    stages {
        stage('Pull Codes from Github'){
            steps{
                checkout scm
            }
        }
        stage('Build Codes by Gradle') {
            steps {
                sh """
                cd ${mainDir}
                ./gradlew clean build
                """
            }
        }
        stage('Build Docker Image by Jib & Push to AWS ECR Repository') {
            steps {
                withEnv(["AWS_PROFILE=sso-dev"]) {
                    sh """
                        # Log in to ECR
                        docker --version
                        docker run -d \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        -v /usr/bin/git:/usr/bin/git \
                        -v /usr/local/bin/aws:/usr/local/bin/aws \
                        jenkins/jenkins:lts
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ecrUrl}
                        # Build and push Docker image
                        cd ${mainDir}
                        pwd
                        ./gradlew jib -Djib.to.image=${ecrUrl}/${repository}:${currentBuild.number} -Djib.console='plain'
                    """
                }
            }
        }
        stage('Deploy to AWS EC2 VM'){
            steps{
                sshagent(credentials : ["${jenkinsAwsKey}"]) {
                    sh """
                    aws --version
                    ssh -o StrictHostKeyChecking=no ${deployAccount}@${deployHost} \
                    'aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ecrUrl}/${repository}; \
                    docker run -d -p 80:8081 -t ${ecrUrl}/${repository}:${currentBuild.number};'
                    """
                }
            }
        }
    }
}
