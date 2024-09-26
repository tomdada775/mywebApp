pipeline {
  agent any

  tools {
    maven 'maven3'
  }

  stages {
    stage('Building first stage one with maven') {
      steps {
        sh 'mvn clean install -f mywebApp/pom.xml'
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

    stage('DEV Deployment fourth stage') {
      steps {
        script {
          echo "Undeploying from DEV Env if already deployed"
          try {
            deploy adapters: [tomcat9(credentialsId: 'f7ae74d8-b13c-4d8b-a34c-89c16b20702c', path: '', url: 'http://ec2-184-73-93-131.compute-1.amazonaws.com:9000/')], contextPath: '/MyWebApp', war: '**/*.war'
          } catch (Exception e) {
            echo "Undeploy failed, possibly because the application is not deployed. Proceeding with deployment."
          }
        }

        echo "Deploying to DEV Env"
        deploy adapters: [tomcat9(credentialsId: 'f7ae74d8-b13c-4d8b-a34c-89c16b20702c', path: '', url: 'http://ec2-184-73-93-131.compute-1.amazonaws.com:9000/')], contextPath: '/MyWebApp', war: '**/*.war'
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
            deploy adapters: [tomcat9(credentialsId: 'f7ae74d8-b13c-4d8b-a34c-89c16b20702c', path: '', url: 'http://ec2-3-89-252-95.compute-1.amazonaws.com:8080/')], contextPath: '/MyWebApp', war: '**/*.war'
          } catch (Exception e) {
            echo "Undeploy failed, possibly because the application is not deployed. Proceeding with deployment."
          }
        }

        echo "Deploying to QA Env"
        deploy adapters: [tomcat9(credentialsId: 'f7ae74d8-b13c-4d8b-a34c-89c16b20702c', path: '', url: 'http://ec2-3-89-252-95.compute-1.amazonaws.com:8080/')], contextPath: '/MyWebApp', war: '**/*.war'
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
  }

  post {
    always {
      echo 'Cleaning up workspace...'
      cleanWs()
    }
  }
}
