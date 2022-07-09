def COLOR_MAP = [
    'SUCCESS':'good',
    'FAILURE': 'danger'
]
pipeline {
    agent any
    environment {
        registryCredential = 'docker-hub1'
        registry = 'shivam2908/vprofileapp'

    }
    stages {

        stage('Build Artifact') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage("Maven Unit Test") {
            steps {
                sh 'mvn test'
            }
        }
        stage("Checkstyle Analysis") {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        
        stage("Build Docker Image") {
            steps {
                script {
                    dockerImage = docker.build registry + ":V${env.BUILD_NUMBER}" 
                }
            }
        }
        
        stage("Push Docker Image") {
            steps {
                script {
                    docker.withRegistry(' ', registryCredential) {
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        
        stage("Deploy Image in Kubernetes through helm charts") {
            agent {label 'KOPS'}
            steps {
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
                }
            }
        
    }
    post {
        always {
            
            echo 'Slack Notification'
            
            slackSend channel: '#top',
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}-:build_id:${env.BUILD_ID}-:build_number:${env.BUILD_NUMBER} \n MORE INFO AT: ${env.BUILD_URL}"
        }
    }
}
