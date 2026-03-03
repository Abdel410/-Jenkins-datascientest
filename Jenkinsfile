pipeline {
environment { // Declaration of environment variables
DOCKER_USER = "abdel411411"
//DOCKER_ID = "abdel411411/datascientestapi" // replace this with your docker-id
DOCKER_IMAGE = "datascientestapi"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
  stage(' Docker Build'){ // docker build image stage
    when {
    branch 'develop'
}
    
    steps {
      script {
      sh '''
        docker rm -f jenkins
        docker build -t $DOCKER_USER/$DOCKER_IMAGE:$DOCKER_TAG .
        sleep 6
      '''
      }
    }
  }

  stage('Docker Push'){ //we pass the built image to our docker hub account
    environment
    {
      DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
    }
    when {
    branch 'develop'
}    
    steps {
      script {
        sh '''
        docker login -u $DOCKER_USER -p $DOCKER_PASS
        docker push $DOCKER_USER/$DOCKER_IMAGE:$DOCKER_TAG

        
        '''
      }
    }
  }


stage('Deploiement en qa'){
  environment {
    KUBECONFIG = credentials("config")
  }
  when {
    branch 'develop'
}  
  steps {
    script {
      sh '''
      mkdir -p .kube
      cp $KUBECONFIG .kube/config
      export KUBECONFIG=$(pwd)/.kube/config

      kubectl create namespace qa || true

      cp fastapi/values.yaml values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

      helm upgrade --install app fastapi --values=values.yml --namespace qa
      '''
    }
  }
}


  stage('Deploiement en staging'){
    when {
    branch 'master'
}    
    environment {
    KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
    }
    steps {
      script {
      sh '''
      rm -Rf .kube
      mkdir .kube
      ls
      cat $KUBECONFIG > .kube/config
      cp fastapi/values.yaml values.yml
      cat values.yml
      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
      helm upgrade --install app fastapi --values=values.yml --namespace staging
      '''
      }
    }
  }
  stage('Deploiement en prod'){
    when {
    branch 'master'
    }   
    environment {
      KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
      }
      steps {
        // Create an Approval Button with a timeout of 15minutes.
        // this require a manuel validation in order to deploy on production environment
        timeout(time: 15, unit: "MINUTES") {
          input message: 'Do you want to deploy in production ?', ok: 'Yes'
        }
        script {
          sh '''
          rm -Rf .kube
          mkdir .kube
          ls
          cat $KUBECONFIG > .kube/config
          cp fastapi/values.yaml values.yml
          cat values.yml
          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
          helm upgrade --install app fastapi --values=values.yml --namespace prod
          '''
        }
      }
    }
  }
}
