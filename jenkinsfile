pipeline {
  agent any

  tools {
    maven 'Maven3'
  }

  stages {
    stage('Building stage one with maven') {
      steps {
        sh 'mvn clean install -f MyWebApp/pom.xml'
      }
    }

    stage('Code Quality with SonarQube second stage') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
        }
      }
    }

    stage('JaCoCo code coverage tool third stage') {
      steps {
        jacoco()
      }
    }

    stage('Slack Notification for Dev Deploy') {
      steps {
        echo "deployed to Dev Env successfully"
        slackSend(channel: 'devopsbuild', message: "Slack Notification for Dev Deploy Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }

    stage('DEV Deployment fourth stage') {
      steps {
        script {
          echo "Undeploying from DEV Env if already deployed"
          try {
            deploy adapters: [tomcat9(credentialsId: '5a45c954-a61d-4643-84c9-498ff0852fac', path: '', url: 'http://ec2-54-227-195-1.compute-1.amazonaws.com:8080/')], contextPath: '/MyWebApp', war: '**/*.war'
          } catch (Exception e) {
            echo "Undeploy failed, possibly because the application is not deployed. Proceeding with deployment."
          }
        }

        echo "Deploying to DEV Env"
        deploy adapters: [tomcat9(credentialsId: '5a45c954-a61d-4643-84c9-498ff0852fac', path: '', url: 'http://ec2-54-227-195-1.compute-1.amazonaws.com:8080/')], contextPath: '/MyWebApp', war: '**/*.war'
      }
    }

    stage('DEV Approve approve stage') {
      steps {
        echo "Taking approval from DEV Manager for QA Deployment"
        timeout(time: 7, unit: 'DAYS') {
          input message: 'Do you want to deploy?', submitter: 'admin'
        }
      }
    }

    stage('QA Deploy qa stage') {
      steps {
        script {
          echo "Undeploying from QA Env if already deployed"
          try {
            deploy adapters: [tomcat9(credentialsId: '5a45c954-a61d-4643-84c9-498ff0852fac', path: '', url: 'http://ec2-54-227-195-1.compute-1.amazonaws.com:8080/')], contextPath: '/MyWebApp', war: '**/*.war'
          } catch (Exception e) {
            echo "Undeploy failed, possibly because the application is not deployed. Proceeding with deployment."
          }
        }

        echo "Deploying to QA Env"
        deploy adapters: [tomcat9(credentialsId: '5a45c954-a61d-4643-84c9-498ff0852fac', path: '', url: 'http://ec2-54-227-195-1.compute-1.amazonaws.com:8080/')], contextPath: '/MyWebApp', war: '**/*.war'
      }
    }

    stage('QA Approve') {
      steps {
        echo "Taking approval from QA manager"
        timeout(time: 7, unit: 'DAYS') {
          input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
        }
      }
    }

    stage('Slack Notification for QA Deploy') {
      steps {
        echo "deployed to QA Env successfully"
        slackSend(channel: 'devopsbuild', message: "Slack Notification for QA successful deployment Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
  }
}
