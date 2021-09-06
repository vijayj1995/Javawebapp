pipeline {
agent any
	stages {
		stage('checkoutscm') {
			steps{
			git credentialsId: '4f89ed93-46cc-4084-a790-f36bd32b36c7', url: 'https://github.com/CHETBSC/Javawebapp.git'
			}
		}
			stage('maven build') {
				steps{
				//cd Javawebapp
					sh 'mvn clean install'
				}
			}
	
	}
}
