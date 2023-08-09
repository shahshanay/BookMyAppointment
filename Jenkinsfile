pipeline {
    environment {
        buildArtifactsPath = "build_artifacts/${currentBuild.number}"
        validationResultsPath = ""
    }
    agent any
    /**
    * Jenkins pipline related variables
    */
    stages {
        // Initialize pipeline
        stage('Initialize') {
            steps {
                script {
                    dockerImageName = "BookMyAppointment/web-app"
                    buildNumberArtifact = "grefId123"

                    /**
                    * DevOps Config App related information
                    * Configuration File information to be uploaded
                    */
                    appName = 'BookMyAppointment'
                    deployableName = 'Production'
                    configFilePath = 'k8s/helm/envs/production/*.yaml'
                    dataFormat = 'yaml'
                }
                echo """---- Build Parameters ----
                applicationName: ${appName}
                """
            }
        }
            
        // Build and publish application image
        stage('Build') {      
            steps {
                checkout scm    
                echo "scm checkout successful"
                
                script {
                    //dockerImageTag = env.BUILD_NUMBER
                    dockerImageTag = "1.2"
                    dockerImageNameTag = "${dockerImageName}" + ":" + "${dockerImageTag}"

                    snDevopsArtifactPayload = '{"artifacts": [{"name": "' + dockerImageName + '",  "version": "' + "${dockerImageTag}" + '", "semanticVersion": "' + "0.1.${dockerImageTag}"+ '","repositoryName": "' + dockerImageName+ '"}, ],"stageName":"Build image","branchName": "main"}'  ;
                    echo "Docker image artifact: ${dockerImageNameTag} "
                    echo "snDevopsArtifactPayload: ${snDevopsArtifactPayload} "

                    snDevOpsArtifact(artifactsPayload:snDevopsArtifactPayload)
                }
            }
        } 
            
        // Validate code and config data
        stage('Validate') {
            parallel {

                stage('Config') {
                    stages('Config Steps') {
                        // Upload configuration data to DevOps Config
                        stage('Upload') {
                            steps {
                                script {
                                    changeSetId = snDevOpsConfigUpload(
                                        applicationName: "${appName}",
                                        target: "deployable",
                                        deployableName: "${deployableName}",
                                        namePath: "web-app-api/v1.0",
                                        configFile: "${configFilePath}",
                                        autoCommit: "true",
                                        autoValidate: "true",
                                        dataFormat: "yaml"
                                    )
                                    echo "Changeset: $changeSetId created"

                                    if(changeSetId == null) {
                                        error "Changeset was not created"
                                    }
                                }
                            }
                            
                        }
                    
                        // Auto-validation was set during upload; get status of snapshot
                        stage('Validate') {
                            steps {
                                script {
                                    changeSetResults = snDevOpsConfigGetSnapshots(
                                        applicationName:"${appName}",
                                        deployableName:"${deployableName}",
                                        changesetNumber:"${changeSetId}",
                                        showResults: false,
                                        markFailed: false
                                    )
                                    if (!changeSetResults) {
                                        echo "No snapshots were created"
                                    } else {
                                        echo "Changeset result : ${changeSetResults}"

                                        def changeSetResultsObject = readJSON text: changeSetResults

                                        changeSetResultsObject.each {
                                            snapshotName = it.name
                                            snapshotObject = it
                                        }
                                        snapshotValidationStatus = snapshotObject.validation
                                        snapshotPublishedStatus = snapshotObject.published 
                                    }
                                }

                                script {
                                    // Set path to snapshot validation results file
                                    validationResultsPath = "${snapshotName}_${currentBuild.projectName}_${currentBuild.number}.xml"
                                    
                                    if(snapshotObject.validation == "passed" || snapshotObject.validation == "passed_with_exception") {
                                        echo "Latest snapshot passed validation"
                                    } else {
                                        error "Latest snapshot failed validation"
                                    }
                                }
                            }
                        }

                        // Publish snapshot now that it passed validation
                        stage('Publish') {
                            when {
                                expression { (snapshotValidationStatus == "passed" || snapshotValidationStatus == "passed_with_exception") && snapshotPublishedStatus == false }
                            }
                            steps {
                                script {
                                    publishSnapshotResults = snDevOpsConfigPublish(applicationName:"${appName}",deployableName:"${deployableName}",snapshotName: "${snapshotName}")
                                }
                            }
                        }
                        
                    }
                }
                
                // Validate application code changes (SIMULATED)
                stage('Code') { 
                    stages {
                        stage('jUnit Test'){ 
                            steps {
                                echo "Running unit tests..."
                            }
                        }
                        
                        stage('SonarQube analysis') {
                            steps {
                                echo "Running code quality analysis..."
                            }
                        }
                    }    
                }
            }
        }

        // Deploy configuration data to UAT environment
        stage('UAT Deployment') {
            steps {
                sleep(time:5,unit:"SECONDS")
            }
        }
        
        // Run functional tests
        stage ('Functional Testing') {
            parallel {      
                stage('Selenium API') { 
                    steps {
                        echo "Selenium API..2..3..4"
                        sleep(time:5,unit:"SECONDS")
                        echo "Selenium API..2..3..4"
                    }
                }
                stage('Selenium UI') {
                    steps {
                        echo "Selenium UI..2..3..4"
                        sleep(time:7,unit:"SECONDS")
                        echo "Selenium API..2..3..4"
                    }
                }
            }
        }

        // Deploy application code and configuration data to production environment
        stage('Deploy to Production') {
                steps {
                    script {
                            echo "Show exported config data from file name"
                            echo " ++++++++++++ BEGIN OF File Content ***************"
                            echo " ++++++++++++ END OF File content ***************"
                            echo "Exported config data handed off to deployment tool"
                            echo "********************** BEGIN Deployment ****************"
                            echo "Applying docker image ${dockerImageNameTag}"
                            echo "********************** END Deployment ****************"
                    }
                }
        }
    }
    // NOTE: attach policy validation results to run (if the snapshot fails validation)
    post {
        always {
            // create tests dir
            //sh "mkdir -p ${buildArtifactsPath}/tests"
            // move policy validation results to build artifacts folder
            //sh "mv ${validationResultsPath} ${buildArtifactsPath}/tests/${validationResultsPath}"
            // attach policy validation results
            echo ">>>>> Displaying Test results <<<<<"
            //junit testResults: "${buildArtifactsPath}/tests/${validationResultsPath}", skipPublishingChecks: true
        }
    }
}
