pipeline {
    agent none
    environment {
        docker_username     = credentials('docker_username')
        docker_password = credentials('docker_password')
    }
    stages {
      stage('Build dokcer image'){
        agent { label 'master' }
        steps{
          sh 'docker build -t todo .'
        }
      }
      stage('lint stage'){
        agent { label 'master' }
        steps{
          sh 'docker run --rm --name todo-lint todo lint'
        }
      }
      stage('unit test stage'){
        agent { label 'master' }
        steps{
          sh 'docker run --rm --name todo-unit-testing todo test:unit'
        }
      }
      stage('E2E test stage'){
        agent { label 'master' }
        steps{
          sh 'docker run --rm --name todo-E2E-testing todo test:e2e --headless'
        }
      }
      stage('push image to docker hub'){
        agent { label 'master' }
        steps{
          sh 'docker login -u $docker_username -p $docker_password'
          sh 'docker tag todo $docker_username/todo_app:${BUILD_NUMBER}'
          sh 'docker push $docker_username/todo_app:${BUILD_NUMBER}'
        }
      }      
      stage('deploy'){
        agent { label 'k8s-master' }
        steps{
          sh 'cat kubernetes/deployment.yml'
          sh 'export IMAGE_NAME=minamaher0/todo_app:$BUILD_NUMBER'
          sh 'cat kubernetes/deployment.yml | envsubst > kubernetes/newDeployment.yml'
          sh 'cat kubernetes/newDeployment.yml'
          sh 'export KUBECONFIG=/home/ubuntu/.kube/config && kubectl apply -f kubernetes/newDeployment.yml'
        }
      }
    }
}