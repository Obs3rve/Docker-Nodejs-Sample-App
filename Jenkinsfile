pipeline {
    agent any
    

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout From SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Obs3rve/Docker-Nodejs-Sample-App.git'
                
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                      def commitID = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                      sh "echo $commitID"
                      sh "BUILD_NUMBER=${BUILD_NUMBER}"
                      sh "docker build -t nodejs:${BUILD_NUMBER} ."
                      sh "docker tag nodejs:${BUILD_NUMBER} chinazao/nodejs:${BUILD_NUMBER} "
                    //   sh "docker tag reddit chinazao/reddit:latest "
                    //   sh "docker push chinazao/reddit:latest "
                      sh "docker push chinazao/nodejs:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                script {
                
                sh "trivy image --format json --severity CRITICAL chinazao/nodejs:${BUILD_NUMBER} > results.json"
                def JSON_FILE="results.json"
                //  Check if the JSON file exists
                    if (!fileExists(JSON_FILE)) {
                        echo "Error: JSON file does not exist."
                        currentBuild.result = 'FAILURE'
                        return
                    }
                   def hasCritical = sh(
                        script: "jq 'any(.Results[]?.Vulnerabilities[]?; .Severity == \"CRITICAL\")' ${JSON_FILE}",
                        returnStdout: true
                    ).trim()
                    sh "cat results.json"
                    
                // # Output the result based on the presence of critical vulnerabilities
                    if (hasCritical == "true") {
                        echo "Critical vulnerabilities found."
                    } else {
                        echo "No critical vulnerabilities found."
                    }
                    
            }
            }
        }
          
      
                stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "Docker-Nodejs-Sample-App"
                GIT_USER_NAME = "Obs3rve"
            }
            steps {
                dir('./') {
                    withCredentials([string(credentialsId: 'jenkins-github', variable: 'GITHUB_TOKEN')]) {
                       sh '''
                            git config user.email "jenkins@gmail.com"
                            git config user.name "jenkins"
                            imageTag=$(grep -oP '(?<=nodejs:)[^ ]+' nodejs-deploy.yaml)
                            echo $imageTag
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
                            sed -i "s/nodejs:${imageTag}/nodejs:${BUILD_NUMBER}/" nodejs-deploy.yaml
                            git add nodejs-deploy.yaml
                            git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''

                    }
                }
            }
        }
        stage('k3s check'){
            steps{
                sh '''
                cat nodejs-deploy.yaml
                kubectl apply -f nodejs-deploy.yaml 
                "cat results.json"
                '''
            }
        }
        
    }
}
