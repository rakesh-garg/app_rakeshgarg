pipeline {
    agent any

    stages {
        stage ('Clean workspace') {
          steps {
            cleanWs()
          }
        }
        stage('Git Checkout'){
            steps {
                git credentialsId: 'GITHUBTOKEN', url: 'https://github.com/rakesh-garg/app_rakeshgarg.git'
            }
        }
        stage('Restore packages') {
          steps {
            bat "dotnet restore ${workspace}\\nagp-devops-us.sln"
          }
        }
        stage('Clean') {
          steps {
              bat "dotnet clean ${workspace}\\nagp-devops-us.sln"
          }
        }
        stage('Build') {
          steps {
              bat "dotnet build ${workspace}\\nagp-devops-us.sln --configuration Release"
          }
        }
        stage('Unit test') {
          steps {
              bat "dotnet test test-project\\test-project.csproj -l:trx;LogFileName=TestProjectTestOutput.xml"
          }
        }
        stage('Publish'){
          steps{
                bat "dotnet publish test-project\\test-project.csproj"
          }
        }
    }
}
