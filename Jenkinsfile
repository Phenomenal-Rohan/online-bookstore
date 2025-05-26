pipeline {
    agent any
    tools {
      dockerTool 'docker'
      jdk 'JAVA_HOME'
      maven 'M2_HOME'
      git 'Default'
    }
    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/DVSR1411/onlinebookstore.git'
            }
        }
        stage('Maven build') {
            steps {
                sh 'mvn clean install package'
            }
        }
        stage('Nexus upload') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'onlinebookstore', classifier: '', file: 'target/onlinebookstore.war', type: 'war']], credentialsId: 'nexus', groupId: 'com.example.myapp', nexusUrl: '3.110.54.27:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'onlinebookstore', version: '0.0.1-SNAPSHOT'
            }
        }
        stage('Docker build') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'passwd', usernameVariable: 'uname')]) {
                    sh 'docker login -u $uname -p $passwd'
                    sh 'docker build -t $uname/onlinebookstore:v1 .'
                }
            }
        }
        stage('Docker push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'passwd', usernameVariable: 'uname')]) {
                    sh 'docker push $uname/onlinebookstore:v1'
                }
            }
        }
        stage('Kubernetes deploy') {
            steps {
                sh 'kubectl apply -f manifests/tomcat.yml'
                sh 'kubectl apply -f manifests/mysql.yml'
            }
        }
    }
}
