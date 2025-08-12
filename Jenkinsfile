pipeline {
    agent any
    tools {
  dockerTool 'DOCKER'
  jdk 'JAVA'
  git 'Default'
  maven 'MAVEN'
}
    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/Phenomenal-Rohan/online-bookstore.git'
            }
        }
        stage('Maven build') {
            steps {
                sh 'mvn clean install package'
            }
        }
        stage('Docker build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'passwd', usernameVariable: 'uname')]) {
                    sh 'docker login -u $uname -p $passwd'
                    sh 'docker build -t $uname/onlinebookstore .'
                }
            }
        }
        stage('Docker push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'passwd', usernameVariable: 'uname')]) {
                    sh 'docker push $uname/onlinebookstore'
                }
            }
        }
        stage('Kubernetes deploy') {
            steps {
                sh '''
                CREDENTIALS=$(aws sts assume-role --role-arn arn:aws:iam::339712783680:role/KubectlRole --role-session-name ec2-kubectl --duration-seconds 900)
                export AWS_ACCESS_KEY_ID="$(echo ${CREDENTIALS} | jq -r '.Credentials.AccessKeyId')"
                export AWS_SECRET_ACCESS_KEY="$(echo ${CREDENTIALS} | jq -r '.Credentials.SecretAccessKey')"
                export AWS_SESSION_TOKEN="$(echo ${CREDENTIALS} | jq -r '.Credentials.SessionToken')"
                export AWS_EXPIRATION=$(echo ${CREDENTIALS} | jq -r '.Credentials.Expiration')
                aws eks update-kubeconfig --name clusterson --region ap-south-1
                kubectl apply -f manifests/.
                '''
              }
           }
        
       }
  }
