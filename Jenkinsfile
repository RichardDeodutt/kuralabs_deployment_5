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
    stage('Docker Build Images') {
      agent{
        label 'Docker'
      }
      steps {
        withCredentials([string(credentialsId: 'DOCKERHUB_USR', variable: 'dockerhub_usr'), 
                      string(credentialsId: 'DOCKERHUB_PWD', variable: 'dockerhub_pwd')]) {
                        dir('intDocker') {
                            sh '''#!/bin/bash
                              docker login --username $dockerhub_usr --password $dockerhub_pwd || exit 1
                              docker build -t $dockerhub_usr/d5-adminer:latest ./adminer || exit 1
                              docker build -t $dockerhub_usr/d5-mysql:latest ./mysql || exit 1
                              docker build -t $dockerhub_usr/d5-nginx:latest ./nginx || exit 1
                              docker build -t $dockerhub_usr/d5-backend:latest ./backend || exit 1
                              docker build -t $dockerhub_usr/d5-frontend:latest ./frontend || exit 1
                              docker push $dockerhub_usr/d5-adminer:latest || exit 1
                              docker push $dockerhub_usr/d5-mysql:latest || exit 1
                              docker push $dockerhub_usr/d5-nginx:latest || exit 1
                              docker push $dockerhub_usr/d5-backend:latest || exit 1
                              docker push $dockerhub_usr/d5-frontend:latest || exit 1
                            '''
                        }
        }
      }
    }
  }
}