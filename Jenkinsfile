pipeline {
	agent {
		kubernetes {
		  defaultContainer 'docker'
		  yamlFile 'pod-template.yaml'
		}
	}

   stages {
      stage('Build and Publish Image') {
         steps {
            //sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=gcr.io/rpa-devops/obi'
            sh 'build -t gcr.io/rpa-devops/obi 12.2.1.4.0'
            sh 'push gcr.io/rpa-devops/obi'
         }
      }
      // Place for new Stage

   } // end of Stages
}
