pipeline{
    agent{
        label "master"
    }
    environment {
        GITLAB_URL = "https://github.com/GMKBabu/dotnet-linux.git"
        GITLAB_CREDENTIALS = "cdb56ac9-d618-4df0-a85f-c41eb9647ef3"
        CUSTOM_BUILD_NUMBER = "DEV-PRD-${BUILD_NUMBER}"
    }
    parameters {
        string (name: 'GITLAB_BRANCH_NAME', defaultValue: 'master', description: 'Git branch to build')
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5', artifactDaysToKeepStr: '3', artifactNumToKeepStr: '1'))
        timeout(time: 60, unit: 'MINUTES')
    }
    stages{
        stage("Checkout"){
            steps{
                script {
                    echo "========executing A========"
                    def scmVars = checkout([$class: 'GitSCM', branches: [[name: "*/${GITLAB_BRANCH_NAME}"]],doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], \
                         userRemoteConfigs: [[credentialsId: "${GITLAB_CREDENTIALS}", url: "${GITLAB_URL}"]]])

                    currentBuild.description = "this is for testing"
                    currentBuild.displayName = "${CUSTOM_BUILD_NUMBER}"
                }
            }
        }
        stage("Build") {
            steps {
                script {
                    echo "====++++executing Source Code Build=++++===="
                    sh "sudo dotnet build ${WORKSPACE}/kk-windows-proj.csproj"

                    echo "====++++executing Source Code Publish++++===="
                    sh "sudo dotnet publish -c Release ${WORKSPACE}/kk-windows-proj.csproj"

                    echo "====++++executing Source Code Zip Publish files++++===="
                    sh "zip -r  ${WORKSPACE}/source.zip ${WORKSPACE}/bin/Release/netcoreapp3.1/publish/*"
                }
            }
        }

        stage("upload artifcat to s3") {
            steps {
                script {
                    s3Upload consoleLogLevel: 'INFO', dontSetBuildResultOnFailure: false, dontWaitForConcurrentBuildCompletion: false, entries: [[bucket: 'babu-connectio/${JOB_NAME}-${BUILD_NUMBER}', excludedFile: '${WORKSPACE}', flatten: false, gzipFiles: false, keepForever: true, managedArtifacts: true, noUploadOnFailure: true, selectedRegion: 'ap-south-1', showDirectlyInBrowser: true, sourceFile: '${WORKSPACE}/source.zip', storageClass: 'STANDARD', uploadFromSlave: false, useServerSideEncryption: false]], pluginFailureResultConstraint: 'FAILURE', profileName: 's3fullaccess', userMetadata: []
                }
            }
        }


    }
}