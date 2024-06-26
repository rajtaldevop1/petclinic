pipeline {
  agent any
  tools {
    maven 'maven3'
    jdk 'JDK8'
  }
  environment {
    AWS_DEFAULT_REGION = 'us-east-1' // Set your AWS region
    CODEARTIFACT_DOMAIN = 'petclinic'    // Set your CodeArtifact domain
    CODEARTIFACT_DOMAIN_OWNER = '381492156727'  // Set your domain owner (AWS account ID)
    CODEARTIFACT_REPO = 'petclinic'        // Set your CodeArtifact repository
  }
  stages {
    stage('Build Maven') {
      steps {
        sh 'pwd'
        sh 'mvn clean install package'
      }
    }
    
    stage('Copy Artifact') {
      steps {
        sh 'pwd'
        sh 'cp -r target/*.jar docker'
      }
    }
    
    stage('Build Docker Image') {
      steps {
        script {
          def customImage = docker.build('rajeshtalla0209/petclinic', "./docker")
          docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            customImage.push("${env.BUILD_NUMBER}")
          }
        }
      }
    }
    
    stage('Upload to CodeArtifact') {
      steps {
        withAWS(credentials: 'aws-credentials-id', region: "${env.AWS_DEFAULT_REGION}") {
          script {
            def artifactFiles = findFiles(glob: 'target/*.jar')
            artifactFiles.each { file ->
              def filePath = file.path
              sh """
                aws codeartifact login --tool npm --repository ${env.CODEARTIFACT_REPO} --domain ${env.CODEARTIFACT_DOMAIN} --domain-owner ${env.CODEARTIFACT_DOMAIN_OWNER}
                aws s3 cp ${filePath} s3://your-s3-bucket/${env.CODEARTIFACT_REPO}/${file.name}
              """
            }
          }
        }
      }
    }
  }
}
