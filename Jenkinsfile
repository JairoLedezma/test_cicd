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
        // creates the template in oCP4 automatically to streamline build
        stage ('Deploying the template') {
            steps {
                script {
                    openshift.withCluster( CLUSTER_NAME ) {
                        openshift.withProject( PROJECT_NAME ){
                            
                            
                            def template = openshift.withProject( PROJECT_NAME ) {
                             // Find the named template and unmarshal into a Groovy object
                                openshift.selector('template','rhdm711-prod-immutable-kieserver').object()
                                }
                            echo "Template contains ${template.parameters.size()} parameters"
                            // This model can be specified as the template to process
                                openshift.create( openshift.process( template,
                                                                "-p", "APPLICATION_NAME=kie=server",
                                                                "-p", "CREDENTIALS_SECRET=credentials",
                                                                "-p", "KIE_SERVER_HTTPS_SECRET=https-keystore",
                                                                "-p", "KIE_SERVER_CONTAINER_DEPLOYMENT=IterationDemo_1.0.0-SNAPSHOT=com.myspace:IterationDemo:1.0.0-SNAPSHOT",
                                                                "-p", "SOURCE_REPOSITORY_URL=https://github.com/JairoLedezma/sample-dmn-iteration.git"
                                                                "-p", "SOURCE_REPOSITORY_REF=master",
                                                                "-p", "CONTEXT_DIR=",
                                                                "-p", "KIE_SERVER_CPU_LIMIT=2",
                                                                "-p", "KIE_SERVER_CPU_REQUEST=300m"))
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

