pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9'
    }
    
    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "localhost:8081"
        NEXUS_REPOSITORY = "maven-releases"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
        GROUP_ID = "com.example"
        ARTIFACT_ID = "springboot-hello"
        VERSION = "1.0.0"
        PACKAGING = "jar"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                git branch: 'main', 
                    url: 'https://github.com/colinbern8/springboot-hello-jenkins.git'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building Spring Boot application...'
                bat 'mvn clean package -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'mvn test'
            }
        }
        
        stage('Code Quality Analysis') {
            steps {
                echo 'Running code quality checks...'
                bat 'mvn verify'
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                echo 'Publishing artifact to Nexus Repository...'
                script {
                    def jarFile = "target/${ARTIFACT_ID}-${VERSION}.${PACKAGING}"
                    def pomFile = "pom.xml"
                    
                    echo "=== Nexus Upload Details ==="
                    echo "Artifact: ${jarFile}"
                    echo "Group ID: ${GROUP_ID}"
                    echo "Artifact ID: ${ARTIFACT_ID}"
                    echo "Version: ${VERSION}"
                    echo "==========================="
                    
                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: GROUP_ID,
                        version: VERSION,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [
                                artifactId: ARTIFACT_ID,
                                classifier: '',
                                file: jarFile,
                                type: PACKAGING
                            ],
                            [
                                artifactId: ARTIFACT_ID,
                                classifier: '',
                                file: pomFile,
                                type: "pom"
                            ]
                        ]
                    )
                    
                    echo "✓ Successfully uploaded ${ARTIFACT_ID}-${VERSION}.${PACKAGING} to Nexus"
                }
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo 'Archiving artifacts in Jenkins...'
                archiveArtifacts artifacts: 'target/*.jar', 
                                 fingerprint: true,
                                 allowEmptyArchive: false
            }
        }
    }
    
    post {
        success {
            echo '=========================================='
            echo '   PIPELINE COMPLETED SUCCESSFULLY!      '
            echo '=========================================='
            echo "Artifact: ${ARTIFACT_ID}-${VERSION}.${PACKAGING}"
            echo "Location: ${NEXUS_URL}/repository/${NEXUS_REPOSITORY}"
            echo '=========================================='
        }
        failure {
            echo '=========================================='
            echo '        PIPELINE FAILED!                 '
            echo '=========================================='
            echo 'Please check the logs above for details.'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}