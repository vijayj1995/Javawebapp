pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo '<--------------- Building --------------->'
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo '<------------- Build completed --------------->'
				}
			}
        }
    }

