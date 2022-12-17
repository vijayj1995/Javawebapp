pipeline {

    // run on jenkins nodes tha has slave label .....

    agent any

    // global env variables

    environment {

        pom = readMavenPom file: "pom.xml"
                appVersion = "pom.version"
                appPomGroupID =  "pom.groupId"
                appGroupID = "appPomGroupID.toString().replace('.', '/')"
                appName = "readMavenPom().getArtifactId()"

    }
    
    stages {
     
        stage('Build') {
            steps {
              // Run the maven build
                sh 'mvn clean install'
                
            }
        }
        stage('Unit Testing junit')
        {
            steps
            {
                       junit 'target/surefire-reports/*.xml'

            }
        }
        stage('Code Coverage')
        {
          steps
          {
         
           jacoco()
           }
        }
        
       stage('Code Quality Check (Sonarqube)')
        {
            environment {
            projectKey = 'Javawebapp'
            projectName = 'Javawebapp'
            projectVersion = '1.1'
            sonarSources = 'src'
            sonarLanguage = 'java'
            sonarBinaries = 'target/classes'
            sonarCoverageformat = '-Dsonar.coverage.jacoco.xmlReportPaths'
            coverageReportsPath = 'target/jacoco.xml'
            sonarSourceEncoding = 'UTF-8'
            }
          steps
          {
             script
             {
               def sonarscanner = tool 'sonar_scanner'
               withSonarQubeEnv('sonarqube') {
               
                    // some block
                    sh """
                    ${sonarscanner}/bin/sonar-scanner -Dsonar.projectKey=${projectKey} \
                        -Dsonar.projectName=${projectName} \
                        -Dsonar.projectVersion=${projectVersion} \
                        -Dsonar.sources=${sonarSources} \
                        -Dsonar.language=${sonarLanguage} \
                        -Dsonar.java.binaries=${sonarBinaries} \
                        ${sonarCoverageformat}=${coverageReportsPath}\
                        -Dsonar.c.file.suffixes=- \
                        -Dsonar.cpp.file.suffixes=- \
                        -Dsonar.objc.file.suffixes=- \
                        -Dsonar.sourceEncoding=${sonarSourceEncoding}
                
                    """
                }
             }
          }
        }
        
       stage('Quality gate') {

            steps {

                timeout(time: 1, unit: 'MINUTES') {

                    retry(3) {

                        script {

                            def qg = waitForQualityGate()

                            if (qg.status != 'OK') {

                                error "Pipeline aborted due to quality gate failure: ${qg.status}"

                            }

                        }

                    }

                }

            }

        }

       stage('Upload to Nexus') {
            steps {
                
               nexusPublisher nexusInstanceId: 'nexus-dev', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/SimpleWebApplication.war']], mavenCoordinate: [artifactId: "${appname}", groupId: "${appPomGroupID}", packaging: 'war', version: "${appVersion}-${BUILD_NUMBER}"]]]
            }
        }
    }
        /*post('Send Email') {
        failure {
            script {
                mail (to: 'wasim.3.akram@bt.com',
                        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) failed",
                        body: "Please visit ${env.BUILD_URL} for further information"
                );
                }
            }
         success {
             script {
                mail (to: 'wasim.3.akram@bt.com', 
                        subject: "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) success.",
                        body: "Please visit ${env.BUILD_URL} for further information.",


                  );
                }
            }      
         }*/
    }
