pipeline{
  agent any
   stages{
    stage('Build Tools') {
      steps {
        sh '''#!/bin/bash
        node --max-old-space-size=100 /usr/bin/npm install --save-dev cypress@7.6.0
        /usr/bin/npx cypress verify
        '''
      }
    }
    agent{label 'Docker'}
    stage('Docker Build Images') {
      steps {
        withCredentials([string(credentialsId: 'DOCKERHUB_USR', variable: 'dockerhub_usr'), 
                      string(credentialsId: 'DOCKERHUB_PWD', variable: 'dockerhub_pwd')]) {
                        dir('intDocker') {
                            sh '''#!/bin/bash
                              docker login --username $dockerhub_usr --password $docker_pwd
                              docker build -t $dockerhub_usr/d5-adminer:latest ./adminer
                              docker build -t $dockerhub_usr/d5-mysql:latest ./mysql
                              docker build -t $dockerhub_usr/d5-nginx:latest ./nginx
                              docker build -t $dockerhub_usr/d5-backend:latest ./backend
                              docker build -t $dockerhub_usr/d5-frontend:latest ./frontend
                              docker push $dockerhub_usr/d5-adminer:latest
                              docker push $dockerhub_usr/d5-mysql:latest
                              docker push $dockerhub_usr/d5-nginx:latest
                              docker push $dockerhub_usr/d5-backend:latest
                              docker push $dockerhub_usr/d5-frontend:latest
                            '''
                        }
        }
      }
    }
  }
}