pipeline {
    agent any
    environment {
        SONAR_HOME = tool "Sonar"
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
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=Swiggy -Dsonar.projectKey=Swiggy"
                }
            }
        }
        stage('OWASP Check'){
            steps{
                echo "OWASP DEPENDANCY CHECK"
            }
        }

        stage('Building Image With Docker') {
            steps {
                sh 'docker build . -t swiggy:latest'
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
        stage('Deploy On K8s') {
            steps {
                sh " cd /home/pro-ubuntu/k8s"
                sh " kubectl apply -f deployment.yml --validate=false "
                sh " kubectl apply -f service.yml --validate=false"
                sh " kubectl port-forward service swiggy-service 3000:80 --address 0.0.0.0 & "
            }
        }

    }
}
