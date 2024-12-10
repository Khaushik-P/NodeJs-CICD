def COLOR_MAP = [
    'FAILURE': 'danger',
    'SUCCESS': 'good'
]

pipeline {
    agent any

    environment {
        APP_NAME = "NodeJs-CICD"
        RELEASE = "1.0.0"
        DOCKER_USER = "khaushik"
        DOCKER_PASS = credentials('docker-hub-credentials')
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {
        stage("Clean Workspace") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    if (env.CHANGE_ID) {
                        echo "Running Pull Request Build for PR #${env.CHANGE_ID}"
                    } else {
                        echo "Running Push Build"
                    }
                    git branch: 'main',
                        credentialsId: 'github',
                        url: 'https://github.com/Khaushik-P/node-application-sample'
                }
            }
        }

        stage('Install Dependencies') {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage("SonarQube Analysis") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-scanner') {
                        sh 'sonar-scanner -Dsonar.projectKey=NodeJs-CICD -Dsonar.sources=. -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${sonar-scanner}'
                    }
                }
            }
        }

        stage("Quality Gate") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-scanner'
                }
            }
        }

        stage("Build and Push to Docker Hub") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                script{
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry('',DOCKER_PASS){
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trivy Image Scan") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                sh 'trivy image ${IMAGE_NAME}:latest > trivyimage.txt'
            }
        }

        stage("Update Deployment Tags") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                sh """
                cat deployment.yaml
                sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                cat deployment.yaml
                """
            }
        }

        stage("Push Changed Deployment File to Git") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                sh """
                git config --global user.name "github-username"
                git config --global user.email "github-email"
                git add deployment.yaml
                git commit -m "Updated Deployment Manifest"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh "git push https://github.com/Khaushik-P/NodeJs-CICD main"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            when {
                expression { !env.CHANGE_ID }
            }
            steps {
                sh """
                kubectl apply -f deployment.yaml
                kubectl rollout status deployment/${APP_NAME}
                """
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications'
            slackSend(
                channel: '#jenkins',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} \n build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
            )
        }
    }
}
