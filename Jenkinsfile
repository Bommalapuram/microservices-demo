pipeline {
    agent any
    
    environment {
        // Docker Hub Details
        DOCKER_HUB_USER = 'devpractice1' 
        IMAGE_NAME      = 'frontend-service'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        
        // SonarQube Details
        SONAR_TOKEN     = 'squ_cf98a98fc60dc11109f16b28b5d018d43d4d74a1'
        SONAR_HOST_URL  = 'http://54.162.179.31:9000/' // ఒకవేళ సోనార్ వేరే ఐపీ మీద ఉంటే ఆ ఐపీ ఇవ్వండి
    }
    
    stages {
        stage('1. Checkout Code') {
            steps {
                echo '📥 Pulling Code from GitHub...'
                checkout scm
            }
        }
        
        stage('2. SonarQube Code Analysis') {
            steps {
                echo '🔍 Starting SonarQube Code Quality Scan...'
                script {
                    // Jenkins build server టెర్మినల్ లో నేరుగా సోనార్ స్కాన్ రన్ చేయడానికి
                    try {
                        sh "sonar-scanner -Dsonar.projectKey=online-boutique-frontend -Dsonar.sources=./src/frontend -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_TOKEN}"
                    } catch (Exception e) {
                        echo "⚠️ Sonar Scanner ٹول సర్వర్ లో లేనట్లు ఉంది, ఐనా బిల్డ్ ముందుకు వెళ్తుంది: ${e.message}"
                    }
                }
            }
        }
        
        stage('3. Docker Build Image') {
            steps {
                echo '🏗️ Building Docker Image for Frontend Service...'
                script {
                    // src/frontend లోపల ఉన్న Dockerfile ని బిల్డ్ చేస్తుంది
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ./src/frontend"
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest ./src/frontend"
                }
            }
        }
        
        stage('4. Docker Push to Registry') {
            steps {
                echo '🚀 Pushing Images to Docker Hub...'
                script {
                    // మీ జెంకిన్స్ 'docker-hub-credentials' ని వాడుకుని లాగిన్ అవుతుంది
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "✨ Success: Docker Image version ${IMAGE_TAG} is live on devpractice1 Docker Hub!"
        }
        failure {
            echo "❌ Error: Pipeline failed. Check console output."
        }
    }
}
