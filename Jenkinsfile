pipeline {
    agent any
    
    stages {
        
        stage("code"){
            steps{
                git url: "https://github.com/hemarawal/node-todo-cicd.git", branch: "master"
                echo 'bhaiyya code clone ho gaya'
            }
        }
         stage("SonarQube Analysis"){
            steps{
               withSonarQubeEnv("Sonar"){
                   sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=nodetodo -Dsonar.projectKey=nodetodo -X"
               }
            }
        }
        stage("SonarQube Quality Gates"){
            steps{
               timeout(time: 1, unit: "MINUTES"){
                   waitForQualityGate abortPipeline: false
               }
            }
        }
         stage("OWASP"){
            steps{
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("build and test"){
            steps{
                sh "docker build -t node-app-test-new ."
                echo 'code build bhi ho gaya'
            }
        }
        stage("Trivy"){
            steps{
                sh "trivy image node-app-batch-6"
            }
        }
        stage("push"){
            steps{
                withCredentials([usernamePassword(credentialsId:"dockerHub",passwordVariable:"dockerHubPass",usernameVariable:"dockerHubUser")]){
                sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                sh "docker tag node-app-test-new:latest ${env.dockerHubUser}/node-app-test-new:latest"
                sh "docker push ${env.dockerHubUser}/node-app-test-new:latest"
                echo 'image push ho gaya'
                }
            }
        }
        stage("deploy"){
            steps{
                sh "docker compose down && docker compose up -d"
                echo 'deployment ho gayi'
            }
        }
    }
}
