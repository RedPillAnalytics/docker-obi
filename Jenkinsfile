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
      stage('Build and Publish Image') {
         steps {
            docker-publish -i redpillanalytics/obi -u $DOCKER_USR -p $DOCKER_PSW -d 12.2.1.4.0
         }
      }
      // Place for new Stage

   } // end of Stages
}
