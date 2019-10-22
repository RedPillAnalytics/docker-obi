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
            sh 'docker build --progress plain -t redpillanalytics/obi 12.2.1.4.0'
         }
      }
      // Place for new Stage

   } // end of Stages
}
