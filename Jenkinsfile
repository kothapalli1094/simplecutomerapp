pipeline {
    agent any

    tools {
        // Must match names configured in Jenkins under "Global Tool Configuration"
        maven "mvn3"
    }

    environment {
        // Nexus configuration
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "54.145.245.39:8081"
        NEXUS_REPOSITORY = "devops"
        NEXUS_CREDENTIAL_ID = "nexus"

        // Sonar Scanner tool name
        SCANNER_HOME = tool 'sonar'
    }

    stages {

        stage("Clone Code") {
            steps {
                git branch: 'feature-1.1', url: 'https://github.com/kothapalli1094/simplecutomerapp.git'
                echo '✅ Repository cloned successfully from feature-1.1 branch'
            }
        }

        stage("Maven Build") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar'
                    withSonarQubeEnv('sonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=Ncodeit \
                                -Dsonar.projectName=Ncodeit \
                                -Dsonar.projectVersion=3.0 \
                                -Dsonar.sources=src \
                                -Dsonar.java.binaries=target/classes \
                                -Dsonar.host.url=http://54.145.245.39:9000
                        """
                    }
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "📦 Found Artifact: ${filesByGlob[0].name} at ${filesByGlob[0].path}"
                    def artifactPath = filesByGlob[0].path

                    if (fileExists(artifactPath)) {
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: pom.version,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                                [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                            ]
                        )
                    } else {
                        error "❌ Artifact not found at ${artifactPath}"
                    }
                }
            }
        }

        stage("Deploy to Tomcat") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat_credentials', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    script {
                        def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                        echo "🚀 Deploying ${warFile} to Tomcat..."
                        sh """
                            curl -u $TOMCAT_USER:$TOMCAT_PASS \
                                -T ${warFile} \
                                "http://54.145.245.39:8080/manager/text/deploy?path=/simplecustomerapp&update=true"
                        """
                    }
                }
            }
        }

        stage("Slack Notification") {
            steps {
                slackSend(
                    channel: "#jenkins-integration",
                    color: "#36a64f",
                    message: "✅ *Simple Customer App* successfully deployed on Tomcat! \nJob: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                )
            }
        }
    }

    post {
        failure {
            slackSend(
                channel: "#jenkins-integration",
                color: "#ff0000",
                message: "❌ Build failed for *${env.JOB_NAME}* #${env.BUILD_NUMBER}. Please check Jenkins logs."
            )
        }
    }
}
