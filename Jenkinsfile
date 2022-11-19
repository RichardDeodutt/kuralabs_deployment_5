pipeline{
  agent any
   stages{
    stage('Build Tools') {
      agent{
        label 'Terraform'
      }
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
    stage('Terraform Init') {
      agent{
        label 'Terraform'
      }
      steps {
        withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'aws_access_key_id'), 
                      string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'aws_secret_access_key')]) {
                        dir('intTerraform') {
                            sh 'terraform init'
                        }
        }
      }
    }
    stage('Terraform Plan') {
      agent{
        label 'Terraform'
      }
      steps {
        withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'aws_access_key_id'), 
                      string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'aws_secret_access_key')]) {
                        dir('intTerraform') {
                            sh 'terraform plan -out plan.tfplan -var="aws_access_key_id=$aws_access_key_id" -var="aws_secret_access_key=$aws_secret_access_key"'
                        }
        }
      }
    }
    stage('Terraform Apply') {
      agent{
        label 'Terraform'
      }
      steps {
        withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'aws_access_key_id'), 
                      string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'aws_secret_access_key')]) {
                        dir('intTerraform') {
                            sh 'terraform apply plan.tfplan'
                            script {
                              env.server_url = "${sh(returnStdout: true, script: 'terraform output -raw alb_url')}"
                            }
                        }
        }
      }
    }
    stage('Cypress E2E') {
      agent{
        label 'Terraform'
      }
      steps {
        sh "echo ${env.server_url} > server_url"
        sh '''#!/bin/bash

          sed -i "s,http://127.0.0.1:5000,$(cat server_url),g" cypress/integration/test.spec.js
          
          StartEpoch=$(date +%s)

          Timeout=300

          Retry=15

          echo "Waiting for Server to come up at: $(cat server_url)"

          while [ $(curl --connect-timeout 1 $(cat server_url) > /dev/null 2>&1 ; echo $?) -ne 0 ]; do

          sleep $Retry

          echo "Checking server..."
          
          if [ $(date +%s) -ge $(echo "$StartEpoch + $Timeout" | bc) ]; then

            echo "Timedout waitig for server" ; exit 1
          
          fi

          done

          echo "Server up!"

          NO_COLOR=1 /usr/bin/npx cypress run --config video=false --spec cypress/integration/test.spec.js
          '''
      }
      post{
        always {
          junit 'test-reports/cypress-results.xml'
        }
      }
    }
    stage('Wait 10 Minutes') {
      steps {
        sh '''#!/bin/bash
          sleep 600
          '''
      }
    }
    stage('Terraform Destroy') {
      agent{
        label 'Terraform'
      }
      steps {
        withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'aws_access_key_id'),
                      string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'aws_secret_access_key')]) {
                        dir('intTerraform') {
                          sh 'terraform destroy -auto-approve -var="aws_access_key_id=$aws_access_key_id" -var="aws_secret_access_key=$aws_secret_access_key"'
                        }
        }
      }
    }
  }
}