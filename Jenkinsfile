pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Fivecoma/petclinic'
            }
        }
        stage('Build') {
            tools {
                maven 'Maven3'
            }
            steps{
                sh 'mvn clean install sonar:sonar'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build --no-cache -t tquatrep/petclinic:1.0.0 /var/lib/jenkins/workspace/${JOB_NAME}"
            }
        }
        stage('Upload to DockerHub') {
            environment {
                PASS = credentials('pass-dockerhub')
            }
            steps {
                sh "docker login -u 'tquatrep' -p \'${PASS}\'"
                sh "docker push tquatrep/petclinic:1.0.0"
            }
        }
        
        stage('Deploy app container') {
            steps {
                sh "ssh vagrant@192.168.60.30 docker stop petclinic"
                sh "ssh vagrant@192.168.60.30 docker container prune -f"
                sh "ssh vagrant@192.168.60.30 docker rmi tquatrep/petclinic:1.0.0"
                sh "ssh vagrant@192.168.60.30 docker run -d -p 80:8080 --name petclinic tquatrep/petclinic:1.0.0"
            }
        }
        
        stage('Deploy to K8s') {
            steps {
                sh 'scp deployment.yml vagrant@192.168.60.60:/home/vagrant'
                sh 'ssh vagrant@192.168.60.60 kubectl apply -f /home/vagrant/deployment.yml'
                sh 'ssh vagrant@192.168.60.60 kubectl delete svc my-service'
                sh 'ssh vagrant@192.168.60.60 kubectl expose deployment petclinic-deployment --type=NodePort --port=8080 --name=my-service'
            }
        }
    }
}
