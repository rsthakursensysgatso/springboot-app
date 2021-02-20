pipeline {
  agent any

  tools {
    jdk 'jdk-11'
    maven 'mvn-3.6.3'
  }

  stages {
    stage('Build') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh "mvn package"
        }
      }
    }

    stage ('OWASP Dependency-Check Vulnerabilities') {
      steps {
        withMaven(maven : 'mvn-3.6.3') {
          sh 'mvn dependency-check:check'
        }

        dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      }
    }

    stage('SonarQube analysis') {
      steps {
        withSonarQubeEnv(credentialsId: 'sonarqube-secret', installationName: 'sonarqube-server') {
          withMaven(maven : 'mvn-3.6.3') {
            sh 'mvn sonar:sonar -Dsonar.dependencyCheck.jsonReportPath=target/dependency-check-report.json -Dsonar.dependencyCheck.xmlReportPath=target/dependency-check-report.xml -Dsonar.dependencyCheck.htmlReportPath=target/dependency-check-report.html'
          }
        }
      }
    }

    stage('Create and push container') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          withMaven(maven : 'mvn-3.6.3') {
            sh "mvn jib:build"
          }
        }
      } 
    }

	stage('Scan Image') {

      steps {
        writeFile file: 'anchore_images', text: 'docker.io/rsthakur83/spring-boot-demo'
        anchore name: 'anchore_images', bailOnFail: false
      }
    }
		

    stage('Deploy to K8s') {
      steps {
          sh 'kubectl apply -f k8s.yaml'
      } 
    }
  }

  post {
    success {
		sh "echo Deployment SUCCESSFUL"
               emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n ----------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                    to: "rsthakur83@gmail.com", 
                    subject: 'Deployment Build SUCCESSFUL : $PROJECT_NAME - #$BUILD_NUMBER'	    
   }

    failure {
	    sh "echo Deployment FAILED"
               emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n ----------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                    to: "rsthakur83@gmail.com", 
                    subject: 'Deployment Build FAILED : $PROJECT_NAME - #$BUILD_NUMBER'	    
    }
}

}
