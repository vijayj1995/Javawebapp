pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo '<--------------- Building --------------->'
                sh 'mvn clean install -Dmaven.test.skip=true'
                echo '<------------- Build completed --------------->'
				}
			}
	 stage('Sonar scan') {
	     steps {
	         withSonarQubeEnv('SonarCloud') {
			 sh 'sonar-scanner'
}
	     }
	 }
        }
    }

