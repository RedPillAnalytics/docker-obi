pipeline {
	agent {
		kubernetes {
		  defaultContainer 'kaniko'
		  yamlFile 'pod-template.yaml'
		}
	}

   stages {
      stage('Build and Publish Image') {
         when { branch "master" }
         steps {
            sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=gcr.io/rpa-devops/obi'
         }
      }
      // Place for new Stage

   } // end of Stages
}
