pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'devpractice1/frontend-service'
        DOCKER_CRED  = 'docker-hub-credentials-id' // Mee Jenkins Docker Hub Credential ID
    }
    stages {
        stage('1. Checkout') {
            steps {
                checkout scm
            }
        }
        stage('2. Build Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }
        stage('3. Login & Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CRED}", passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                    sh "echo ${PASSWORD} | docker login -u ${USERNAME} --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                }
            }
        }
        // ********** ఇదో ఆటోమేషన్ కోసం మనం యాడ్ చేయాల్సిన అసలైన స్టేజ్ **********
        stage('4. Update Helm Manifest') {
            steps {
                script {
                    // values.yaml లో ఉన్న పాత ఇమేజ్ ట్యాగ్‌ను కొత్త Jenkins Build Number తో రీప్లేస్ చేస్తుంది
                    sh "sed -i 's/tag: .*/tag: \"${BUILD_NUMBER}\"/' helm-chart/values.yaml"
                    
                    // అప్‌డేట్ అయిన values.yaml ని తిరిగి మీ GitHub Repo కి పుష్ చేస్తుంది
                    withCredentials([usernamePassword(credentialsId: 'git-cred', passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]) {
                        sh '''
                            git config user.email "jenkins@devops.com"
                            git config user.name "Jenkins CI"
                            git add helm-chart/values.yaml
                            git commit -m "chore: automated image tag update to v${BUILD_NUMBER} [skip ci]"
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/Bommalapuram/microservices-demo.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
