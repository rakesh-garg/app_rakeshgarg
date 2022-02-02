pipeline {
    agent any
    
     environment {
        scannerHome = tool name: 'sonar_scanner_dotnet'
    }

    stages {
        stage('Restore packages') {
          steps {
            cleanWs()
            checkout([$class: 'GitSCM', branches: [[name: '*/developer']], extensions: [], userRemoteConfigs: [[credentialsId: 'GITHUBTOKEN', url: 'https://github.com/rakesh-garg/app_rakeshgarg.git']]])
          }
        }
        stage('Start sonarQube analysis'){
            steps{
                echo "Start SonarQube Analysis"
                withSonarQubeEnv("Test_Sonar") {
                  bat "dotnet sonarscanner begin /k:Sonar-RakeshGarg -d:sonar.cs.opencover.reportsPaths=test-project/coverage.opencover.xml -d:sonar.cs.xunit.reportsPaths='test-projects/TestProjectTestOutput.xml'"   
                }
            }
        }
        stage('Code build') {
          steps {
              bat "dotnet restore ${workspace}\\nagp-devops-us.sln"
              bat "dotnet clean ${workspace}\\nagp-devops-us.sln"
              bat "dotnet build ${workspace}\\nagp-devops-us.sln --configuration Release"
          }
        }
        stage('Test case execusion') {
          steps {
            bat "dotnet test ${workspace}\\test-project\\test-project.csproj /p:CollectCoverage=true /p:CoverletOutputFormat=opencover -l:trx;LogFileName=TestProjectTestOutput.xml"
          }
        }
        stage('Stop SonarQube Analysis') {
           steps {
            withSonarQubeEnv("Test_Sonar") { 
                bat "dotnet sonarscanner end"
            }
           }
        }
    }
}
