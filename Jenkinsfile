pipeline {
    agent any

    environment {
        // Your existing credential ID
        
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
        
        // YOUR Docker Hub Username
        DOCKERHUB_USER = "aayushbhargav57" 
        
        // Image Names
        BACKEND_IMAGE = "${DOCKERHUB_USER}/mlops-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/mlops-frontend"
        DOCKER_TAG = "latest"
        
        // Email for notifications
        EMAIL_ID = "aayushbhargav0507@gmail.com"
    }

    stages {
        stage('Checkout') {
            steps {
                // Pulls code from the branch that triggered the webhook
                checkout scm
            }
        }

        stage('Train Model (CI)') {
            steps {
                echo 'Pulling REAL Data from DVC...'
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                
                # 1. CLEANUP
                rm -rf mlruns
                rm -f mlflow.db
                # Note: We do NOT remove the 'data' folder, DVC needs it.
                
                # 2. CONFIGURE DVC (The Strong Connection)
                # We tell Jenkins where the "Storage Locker" is.
                # Jenkins uses the exact same folder your laptop uses.
                dvc remote add -d -f mylocal /tmp/dvc_store
                
                # 3. PULL DATA
                # This downloads 'data/train.csv' from the storage locker.
                # It will be the EXACT version you pushed from your laptop.
                dvc pull
                
                # 4. TRAIN
                python3 train.py
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Images (With Model Baked In)...'
                // The Dockerfile now copies the 'mlruns' folder we just created!
                sh "docker build -f Dockerfile.backend -t ${BACKEND_IMAGE}:${DOCKER_TAG} ."
                sh "docker build -f Dockerfile.frontend -t ${FRONTEND_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // Secure login using your existing credentials syntax
                sh """
                echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                
                # Push Backend
                docker push ${BACKEND_IMAGE}:${DOCKER_TAG}
                
                # Push Frontend
                docker push ${FRONTEND_IMAGE}:${DOCKER_TAG}
                
                docker logout
                """
            }
        }

        stage('Update Kubernetes') {
            steps {
                echo 'Updating K8s Deployment...'
                // We point kubectl explicitly to the config file we just copied
                sh "kubectl --kubeconfig=/var/lib/jenkins/kubeconfig rollout restart deployment/backend-deployment"
                sh "kubectl --kubeconfig=/var/lib/jenkins/kubeconfig rollout restart deployment/frontend-deployment"
            }
        }
    }

    post {
        success {
            mail bcc: '',
                 body: "SUCCESS: MLOps Pipeline (Build ${BUILD_NUMBER}) deployed new AI models to Docker Hub.",
                 from: 'jenkins@localhost',
                 subject: "Pipeline SUCCESS: MLOps Project Build #${BUILD_NUMBER}",
                 to: "${EMAIL_ID}"
        }
        failure {
            mail bcc: '',
                 body: "FAILURE: MLOps Pipeline (Build ${BUILD_NUMBER}) crashed. Check logs.",
                 from: 'jenkins@localhost',
                 subject: "Pipeline FAILURE: MLOps Project Build #${BUILD_NUMBER}",
                 to: "${EMAIL_ID}"
        }
        always {
            cleanWs()
        }
    }
}