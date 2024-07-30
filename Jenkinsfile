pipeline {
    agent any

    environment {
        ARTIFACTORY_SERVER = 'jfrog-cloud-trail-instance' // The ID of your Artifactory server configured in Jenkins
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
                    def server = Artifactory.server(ARTIFACTORY_SERVER)
                    def rtMaven = Artifactory.newMavenBuild()
                    
                    // Configure resolver and deployer
                    rtMaven.resolver(server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot')
                    rtMaven.deployer(server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local')
                    rtMaven.tool = MAVEN_TOOL

                    // Run Maven build
                    def buildInfo = rtMaven.run(pom: 'pom.xml', goals: 'clean install')

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
