pipeline {
    agent any

    environment {
        ARTIFACTORY_SERVER = 'jfrog-cloud-trial-instance' // Ensure this matches the configured server ID
        MAVEN_TOOL = 'M3' // The name of the Maven tool configured in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from your source control
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Setting up Artifactory server"
                    def server = Artifactory.server(ARTIFACTORY_SERVER)
                    echo "Artifactory server set up: ${server}"

                    echo "Setting up Maven build"
                    def rtMaven = Artifactory.newMavenBuild()
                    rtMaven.tool = MAVEN_TOOL
                    
                    // Configure resolver and deployer
                    rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
                    rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
                    
                    // Run Maven build
                    def buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install'
                    echo "Maven build completed: ${buildInfo}"

                    // Publish build information and artifacts to Artifactory
                    server.publishBuildInfo(buildInfo)
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Build and artifact push to Artifactory completed successfully.'
        }
        failure {
            echo 'Build or artifact push to Artifactory failed.'
        }
    }
}
