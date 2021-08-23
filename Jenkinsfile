pipeline {
    agent any
    tools{
        oc 'oc'
        maven 'maven-3.6.3'
        jdk 'jdk11'
    }
    stages {
        stage('Fetching Git Repository') {
            steps {
                git url: GIT_URL, branch: BRANCH
            }
        }
        stage ('Artifactory configuration') {
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "localhost-jfrog-server",
                    releaseRepo: "libs-release-local",
                    snapshotRepo: "libs-snapshot-local"
                )
            }
        }
        stage ('Maven Build') {
            steps {
                rtMavenRun (
                    tool: "maven-3.6.3",
                    pom: 'pom.xml',
                    goals: '-s settings.xml clean install',
                    deployerId: "MAVEN_DEPLOYER"
                )
            }
        }
        stage ('Running Unit Tests') {
            steps {
                rtMavenRun (
                    tool: "maven-3.6.3",
                    pom: 'pom.xml',
                    goals: '-s settings.xml test'
                )
            }
        }
        stage ('Making Namespace') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        if( NEW_PROJECT==true){
                            def projectCommand = "oc new-project "+PROJECT_NAME+" "+TOKEN
                            sh projectCommand
                        }
                    }
                }  
            }
        }
        
       stage ('Create & Replace Configurations') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def processedTemplate
                            
                            // if the new_project box is checked then a fresh install of the necessary files is ran
                            // otherwise, you could change the files in template-replace and then run it again to update
                            if( NEW_PROJECT ){
                                 try {
                                    processedTemplate = openshift.process( "-f", "./templates/template-create.yaml", "--param-file=./templates/template-create.env")
                                    def createResources = openshift.create( processedTemplate )
                                    createResources.logs('-f')
                                 } catch (err) {
                                    echo err.getMessage()
                                }
                            } else{
                                try {
                                    processedTemplate = openshift.process( "-f", "./templates/template-replace.yaml", "--param-file=./templates/template-replace.env")
                                    def replaceResources = openshift.replace( processedTemplate )
                                    replaceResources.logs('-f')
                                 } catch (err) {
                                    echo err.getMessage()
                                }
                            }
                            
                        }
                    }
                }
            }
        }
    
         
        stage ('Uploading Artifacts to Artifactory') {
            steps {
                rtPublishBuildInfo (
                    serverId: "localhost-jfrog-server"
                )
            }
        }
        stage ('Building and Pushing Image to Quay') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def buildConfig = openshift.selector( 'buildconfig/' + BUILD_CONFIG )
                            buildConfig.startBuild()
                            buildConfig.logs('-f')
                        }
                    }
                }    
            }
        }
        stage ('Deploying') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            def devConfig = openshift.selector( 'deploymentconfig/' + DEPLOYMENT_CONFIG )
                            devConfig.rollout().latest()
                        }
                    }
                }    
            }
        }
    }

}
