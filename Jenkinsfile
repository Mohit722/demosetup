pipeline {
  agent none

  options {
    timeout(time: 15, unit: 'MINUTES')
  }

  parameters {
    string(name: 'ECRURL', defaultValue: 'your-ecr-url', description: 'ECR repository URL')
    string(name: 'REPO', defaultValue: 'wezvabaseimage', description: 'Name of the Docker repository')
    string(name: 'REGION', defaultValue: 'ap-south-1', description: 'AWS region')
  }

  environment {
    AWS_CREDENTIALS_ID = 'aws_credentials' // Replace with your AWS credentials ID in Jenkins
  }

  stages {
    stage('Checkout') {
      agent { label 'demo' }
      steps {
        git branch: 'master', url: 'https://github.com/Mohit722/demosetup.git'
      }
    }

    stage('Build Image') {
      agent { label 'demo' }
      steps {
        script {
          // Prepare the Tag name for the image
          def dockerTag = "${params.REPO}:${env.BUILD_ID}"
          docker.withRegistry("${params.ECRURL}", AWS_CREDENTIALS_ID) {
            // Build Docker image
            def myImage = docker.build(dockerTag)

            // Push the Image to the Registry
            myImage.push()
          }
        }
      }
    }

    stage('Scan Image') {
      agent { label 'demo' }
      steps {
        withAWS(credentials: AWS_CREDENTIALS_ID, region: "${params.REGION}") {
          sh "./getimagescan.sh ${params.REPO} ${env.BUILD_ID} ${params.REGION}"
        }
      }
      post {
        always {
          sh "docker rmi ${params.REPO}:${env.BUILD_ID}"
        }
      }
    }
  }
}

