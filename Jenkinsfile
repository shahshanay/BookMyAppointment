pipeline {
    environment {
        buildArtifactsPath = "build_artifacts/${currentBuild.number}"
        validationResultsPath = ""
    }
    agent any

    stages {
        // Build and publish application image
        stage('Build') {      
            steps {
                checkout scm    
                echo "scm checkout successful"
                
                script {
                    //dockerImageTag = env.BUILD_NUMBER
                    dockerImageName = "TestMyApp/web-app"
                    buildNumberArtifact = "grefId123"
                    dockerImageTag = "1.2"
                    dockerImageNameTag = "${dockerImageName}" + ":" + "${dockerImageTag}"

                    // snDevopsArtifactPayload = '{"artifacts": [{"name": "' + dockerImageName + '",  "version": "' + "${dockerImageTag}" + '", "semanticVersion": "' + "0.1.${dockerImageTag}"+ '","repositoryName": "' + dockerImageName+ '"}, ],"stageName":"Build image","branchName": "main"}'  ;
                    // echo "Docker image artifact: ${dockerImageNameTag} "
                    // echo "snDevopsArtifactPayload: ${snDevopsArtifactPayload} "

                    // snDevOpsArtifact(artifactsPayload:snDevopsArtifactPayload)
                }
            }
        } 
            
        // Validate Test
        stage('Test') {
            parallel {
                stage('Code') {
                    stages('Code Steps') {
                        // Validate application code changes (SIMULATED)
                        stage('jUnit Test') { 
                            steps {
                                echo "Running unit tests..."
                            }
                        }
                        
                        stage('Selenium Test') { 
                            steps {
                                echo "Selenium API..2..3..4"
                                sleep(time:5,unit:"SECONDS")
                                
                                echo "Selenium UI..2..3..4"
                            }
                        }
                    }   
                }

                stage('Config') {
                    stages('Config Steps') {
                        stage ("Upload") {
                            steps {
                                script {
                                    changeSetResults = snDevOpsConfig(
                                         applicationName: 'BookMyAppointment',
                                         target: 'deployable',
                                         deployableName: 'Production_US_EAST',
                                         namePath: 'helmchart',
                                         configFile: 'k8s/helm/envs/prod_us_east/*',
                                         dataFormat: 'yaml',
                                         autoCommit: 'true',
                                         autoValidate: 'true',
                                         autoPublish: 'true'
                                    )

                                    if (changeSetResults == null) {
                                        echo "No snapshots were created"
                                    }
                                    else {
                                        echo "ChangeSetResults: ${changeSetResults}"

                                        def changeSetResultsObject = readJSON text: changeSetResults

                                        changeSetResultsObject.each {
                                            snapshotName = it.name
                                            snapshotValidationStatus = it.validation

                                            // Set path to snapshot validation results file
                                            validationResultsPath = "${snapshotName}*${currentBuild.number}.xml"
                                            // attach policy validation results
                                            junit testResults: "${validationResultsPath}", skipPublishingChecks: true
                                            
                                            if (snapshotValidationStatus == "failed") {
                                                error "Lastest validation failed for ${snapshotName}"
                                            }
                                            
                                        }
                                        
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }                    

        

        stage ('Deploy') {
            // Deploy application code and configuration data to production 1 environment
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
            
                // Deploy application code and configuration data to production 2 environment
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

    

}
