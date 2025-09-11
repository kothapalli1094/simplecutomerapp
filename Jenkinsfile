pipeline {
    agent any
    tools {
        // Note: this should match with the tool name configured in your jenkins instance (JENKINS_URL/configureTools/)
        maven "MVN_HOME"
        
    }
	 environment {
        // This can be nexus3 or nexus2
        NEXUS_VERSION = "nexus3"
        // This can be http or https
        NEXUS_PROTOCOL = "http"
        // Where your Nexus is running
        NEXUS_URL = "3.92.191.162:8081"
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = "Declarative-app"
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = "Nexus_server"
	SCANNER_HOME = tool 'SonarQube'
    }
    stages {
        stage("clone code") {
            steps {
                script {
                    // Let's clone the source
                    git 'https://github.com/kothapalli1094/simplecutomerapp.git';
					echo '✅ Repository cloned successfully from feature-1.1 branch'
                }
            }
        }
		
        stage("mvn build") {
            steps {
                script {
                    // If you are using Windows then you should use "bat" step
                    // Since unit testing is out of the scope we skip them
                    sh 'mvn -Dmaven.test.failure.ignore=true clean install'
                }
            }
        }
	stage('SonarCloud') {
            steps {
                withSonarQubeEnv('SonarQube') {
				sh '$SCANNER_HOME/bin/sonar-scanner \
				-Dsonar.projectKey=Ncodeit \
				-Dsonar.projectName=Ncodeit \
				-Dsonar.projectVersion=2.0 \
				-Dsonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/src/ \
				-Dsonar.binaries=target/classes/com/visualpathit/account/controller/ \
				-Dsonar.junit.reportsPath=target/surefire-reports \
				-Dsonar.jacoco.reportPath=target/jacoco.exec \
				-Dsonar.java.binaries=src/com/room/sample '
				
		     }
		  }
	    }
        stage("publish to nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml";
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path;
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
			    groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                // Lets upload the pom.xml file for additional information for Transitive dependencies
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
		stage("Deploy to Tomcat") {
    steps {
        withCredentials([usernamePassword(credentialsId: 'tomcat', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
            script {
                // Find the WAR file built by Maven
                def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()

                echo "Deploying ${warFile} to Tomcat at context path /simplecustomerapp ..."

                sh """
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T ${warFile} \
                         "http://54.91.10.42:8080/manager/text/deploy?path=/simplecustomerapp&update=true"
        
                        """
                    }
                }
            }
		}
        
		 stage("Slack Notification") {
            steps {
                slackSend(
                    channel: "${SLACK_CHANNEL}",
                    color: "#36a64f",
                    message: "Declarative pipeline for *Simple Customer App* has been successfully deployed in Tomcat ✅ by SNL for Job: ${env.JOB_NAME} [${env.BUILD_NUMBER}]"
                )
            }
        }
    }
}
