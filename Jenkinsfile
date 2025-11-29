pipeline {
  agent any

tools {
    nodejs "Nodejs"
}

 environment {
    DOCKER_REGISTRY = "docker.io"
    DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_CREDENTIAL')
    VERSION = "${env.BUILD_ID}"
  }

  stages {

 stage('Install Dependencies') {
      steps {

        sh 'npm ci'
      }
    }

    stage('Build Project') {
      steps {
        // Build the Angular project
        sh 'npm run build'
      }
    }


    stage('Docker Build and Push') {
      steps {
          sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
          sh 'docker build -t nguyendinhvu/angular-server:${VERSION} .'
          sh 'docker push nguyendinhvu/angular-server:${VERSION}'
      }
    }


     stage('Cleanup Workspace') {
      steps {
        deleteDir()
      }
    }

     stage('Update Image Tag in GitOps') {
        steps {
            script {
                sh """
                    rm -rf deployment-service
                    git clone https://${GITHUB_TOKEN}@github.com/NguyenDinhVu2003/deployment-service.git
                    cd deployment-service
                    git config user.email "jenkins@ci.com"
                    git config user.name "Jenkins CI"
                    sed -i "s|image:.*|image: nguyendinhvu/angular-server:${VERSION}|" aws/angular-manifest.yml
                    git add .
                    git commit -m "Update image tag to ${VERSION}"
                    git push
                """
            }
        }
    }
  }

}


