pipeline {
	agent {
		kubernetes {
		  defaultContainer 'gradle'
		  yamlFile 'pod-template.yaml'
		}
	}

   stages {
      stage('Build and Publish Image') {
         steps {
            //sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --cache=true --destination=gcr.io/rpa-devops/obi'
            sh 'ls -l'
         }
      }
      // Place for new Stage

   } // end of Stages
}
