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
        stage("build") {
            parallel {
                stage("Build code") {
                    steps {
                        script {
                                echo "========executing Source Code Build========"
                                sh "dotnet build ${WORKSPACE}/kk-windows-proj.csproj"
                        }
                    }
                }
                stage("Build Publish") {
                    steps {
                        script {
                                echo "========executing Source Code Publish========"
                                sh "dotnet publish -c Release ${WORKSPACE}/kk-windows-proj.csproj"
                        }
                    }
                }
                stage("Zip Publish files") {
                    steps {
                        script {
                            echo "========executing Source Code Build========"
                            sh '''
                                zip -r  ${WORKSPACE}/source.zip ${WORKSPACE}/bin/Release/netcoreapp3.1/publish/*
                            '''
                        }
                    }
                }
            }
        }

    }
}