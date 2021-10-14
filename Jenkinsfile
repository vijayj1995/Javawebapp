pipeline {

    // run on jenkins nodes tha has slave label .....

    agent { label 'maven' }

    // global env variables

    /*environment {

        EMAIL_RECIPIENTS = 'akshay.kg@bt.com'

    }*/
    
    stages {
        stage ('checkout scm') {
            steps {
        updateGitlabCommitStatus name: STAGE_NAME, state: 'running'
                pipelineStage = "${STAGE_NAME}"
                step([$class: 'WsCleanup'])
                checkout scm
                branchName = "${BRANCH_NAME}"
                commitID = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
                commiterName = sh(returnStdout: true, script: 'git show --format="<%aE>" HEAD | head -1').trim().toString().replace('<', '').replace('>', '').trim()
                totalCommitsOfUser = sh(returnStdout: true, script: "git shortlog -s -n -e --all --no-merges | grep -i ${commiterName} | sed -e 's/ //g' | cut -f 1 | head -1").trim()
                totalCommitsInBranch = sh(returnStdout: true, script: 'git rev-list --count HEAD').trim()
                updateGitlabCommitStatus name: STAGE_NAME, state: 'success'
            }
        }
        stage('Build') {
            steps {
                pom = readMavenPom file: 'pom.xml'
            appVersion = pom.version

            if ("${branchName}" == 'Dev' || "${branchName}" == 'Test' || "${branchName}" == 'dev') {
                appVersion = "${appVersion}" =~ '-SNAPSHOT' ? pom.version : "${appVersion}" + '-SNAPSHOT'
            } else {
                appVersion = "${appVersion}" =~ '.RELEASE' ? pom.version : "${appVersion}" + '.RELEASE'
            }

            appPomGroupID = pom.groupId
            appGroupID = appPomGroupID.toString().replace('.', '/')
            appName = pom.artifactId
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
                // Deploy to Nexus
               nexusPublisher nexusInstanceId: 'nexus-dev', nexusRepositoryId: 'maven-releases', packages: []
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
