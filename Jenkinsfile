pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
        GITHUB_TOKEN = credentials('GithubToken')  // Use a Jenkins credential for GitHub PAT
        GITHUB_USERNAME = 'Bakhtawarkhan90'   // Replace with your GitHub username

    }
    stages {
        stage('Code Clone') {
            steps {
                git url: 'https://github.com/Bakhtawarkhan90/Swiggy.git', branch: 'master'
                
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy -Dsonar.projectKey=Swiggy -X"
                }
            }
        }
        
        stage('Building Image With Docker') {
            steps {
                sh "docker build . -t swiggy:latest"
            }
        }

        stage('Trivy Image Scanning') {
            steps {
                sh "trivy image swiggy:latest"
            }
        }

        stage('Push To Docker-Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh "echo \$dockerHubPass | docker login -u \$dockerHubUser --password-stdin"
                    sh "docker tag swiggy:latest ${env.dockerHubUser}/swiggy:latest"
                    sh "docker push ${env.dockerHubUser}/swiggy:latest"
                }
            }
        }

        stage('OWASP DC') {
            steps {
                  dependencyCheck additionalArguments: '--scan .', odcInstallation: 'OWASP'
                  dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }


        stage('Deploy On K8s') {
            steps {
                sh 'cd /var/lib/jenkins/workspace/Swiggy/K8s'
                sh 'kubectl delete -f .'
                sh 'kubectl apply -f . '
                sh 'kubectl port-forward svc/swiggy-service 8000:80 --address 0.0.0.0 &'
            }
        }
    }
}
