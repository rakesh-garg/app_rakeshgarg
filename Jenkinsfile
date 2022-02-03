pipeline {
    agent any
    
     environment {
        scannerHome = tool name: 'sonar_scanner_dotnet'
        userName = 'rakeshgarg'
        applicationName = 'app_rakeshgarg'
        dockerHubRepository ='rakeshgarg2021/app_rakeshgarg'
    }

    stages {
        stage('Nuget restore') {
          steps {
            // checkout([$class: 'GitSCM', branches: [[name: '*/developer']], extensions: [], userRemoteConfigs: [[credentialsId: 'GITHUBTOKEN', url: 'https://github.com/rakesh-garg/app_rakeshgarg.git']]])
              echo "Nuget restore - dotnet restore ${workspace}\\nagp-devops-us.sln"
              bat "dotnet restore ${workspace}\\nagp-devops-us.sln"
          }
        }
        stage('Start sonarQube analysis'){
            when { branch "master" }
            steps{
                echo "Start sonarQube analysis for master branch only"
                withSonarQubeEnv("Test_Sonar") {
                  bat "dotnet sonarscanner begin /k:sonar-${userName} -d:sonar.cs.opencover.reportsPaths=test-project/coverage.opencover.xml -d:sonar.cs.xunit.reportsPaths='test-projects/TestProjectTestOutput.xml'"   
                }
            }
        }
        stage('Code build') {
          steps {
              echo "Code build - dotnet build ${workspace}\\nagp-devops-us.sln --configuration Release"
              bat "dotnet clean ${workspace}\\nagp-devops-us.sln"
              bat "dotnet build ${workspace}\\nagp-devops-us.sln --configuration Release"
          }
        }
        stage('Test case execusion') {
          steps {
              echo "Test case execution - dotnet test ${workspace}\\test-project\\test-project.csproj /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestProjectTestOutput.xml"
              bat "dotnet test ${workspace}\\test-project\\test-project.csproj /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestProjectTestOutput.xml"
          }
        }
        stage('Stop sonarqube analysis') {
            when { branch "master"}
           steps {
            withSonarQubeEnv("Test_Sonar") { 
                echo "Stop sonarqube analysis (master only) - dotnet sonarscanner end"
                bat "dotnet sonarscanner end"
            }
           }
        }
        
        stage ("Release artifact") {
            steps {
                echo "Release artifact - dotnet publish -c Release -o ${applicationName}/app/${userName}"
                bat "dotnet publish -c Release -o ${applicationName}/app/${userName}"
            }
        }
        
        stage ("Docker build image") {
            steps {
                echo "Docker build image - docker build -t i-${userName}-${BRANCH_NAME}:${BUILD_NUMBER} --no-cache -f Dockerfile ."
                bat "docker build -t i-${userName}-${BRANCH_NAME}:${BUILD_NUMBER} --no-cache -f Dockerfile ."
            }
        }
        stage ("Docker push image") {
            steps {
                withDockerRegistry([ credentialsId: "dockerhub-RakeshGarg-Token", url: "" ]) {
                     bat "docker tag i-${userName}-${BRANCH_NAME}:${BUILD_NUMBER} ${dockerHubRepository}:i-${userName}-${BRANCH_NAME}-${BUILD_NUMBER}"
                     bat "docker tag i-${userName}-${BRANCH_NAME}:${BUILD_NUMBER} ${dockerHubRepository}:i-${userName}-${BRANCH_NAME}-latest"
                     bat "docker push ${dockerHubRepository}:i-${userName}-${BRANCH_NAME}-${BUILD_NUMBER}"
                     bat "docker push ${dockerHubRepository}:i-${userName}-${BRANCH_NAME}-latest"
                }
            }
        }
        
        stage ("Docker deploy image") {
            steps {
                echo "Docker deploy image - docker run --name c-${userName}-${BRANCH_NAME} -d -p ${port}:80 ${dockerHubRepository}:i-${userName}-${BRANCH_NAME}-latest"
                bat "docker run --name c-${userName}-${BRANCH_NAME} -d -p ${port}:80 ${dockerHubRepository}:i-${userName}-${BRANCH_NAME}-latest"
            }
        }
    }
}
