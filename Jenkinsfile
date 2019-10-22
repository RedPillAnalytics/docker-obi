pipeline {
	agent {
		kubernetes {
		  defaultContainer 'docker'
		  yamlFile 'pod-template.yaml'
		}
	}
   environment {
      DOCKER = credentials('docker-hub')
   }

   stages {
      stage('Build') {
         steps {
            sh 'docker build --progress plain -t redpillanalytics/obi 12.2.1.4.0'
         }
      stage('Publish') {
         when { branch "master" }
         steps {
            sh "docker login -u ${DOCKER_USR} -p ${DOCKER_PSW}"
            sh "docker push redpillanalytics/obi:12.2.1.4.0"
            sh "docker push redpillanalytics/obi:latest"
         }
      }
      // Place for new Stage

   } // end of Stages
}
