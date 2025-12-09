pipeline {
    agent any

    environment {
        // Existing credential ID
        
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
    
        // YOUR Docker Hub Username
        DOCKERHUB_USER = "aayushbhargav57" 
        
        // Image Names
        BACKEND_IMAGE = "${DOCKERHUB_USER}/mlops-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/mlops-frontend"
        DOCKER_TAG = "latest"
        
        // Email for notifications
        EMAIL_ID =  "aayushbhargav0507@gmail.com"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // stage('Configure Remote Host') {
        //     steps {
        //         echo 'Running Configuration Management playbook (install Docker/K8s tools)...'

        //         withCredentials([
        //             string(credentialsId: 'sudo_pass_vault_credentials', variable: 'ANSIBLE_VAULT_PASSWORD')
        //         ]) {
        //             sh '''
        //                 # Write vault password WITHOUT Groovy interpolation
        //                 printf "%s" "$ANSIBLE_VAULT_PASSWORD" > vault-pass.txt

        //                 ansible-playbook \
        //                     -i ansible/inventory.ini \
        //                     ansible/playbook-1.yml \
        //                     --vault-password-file vault-pass.txt \
        //                     --extra-vars workspace="$WORKSPACE"

        //                 rm -f vault-pass.txt
        //             '''
        //         }
        //     }
        // }

        // stage('Build, Train, & Deploy (Remote)') {
        //     steps {
        //         echo 'Executing full CI/CD pipeline on Ansible host...'

        //         withCredentials([
        //             usernamePassword(credentialsId: 'dockerhub_credentials',
        //                             usernameVariable: 'DOCKER_USR',
        //                             passwordVariable: 'DOCKER_PSW'),
        //             string(credentialsId: 'sudo_pass_vault_credentials', variable: 'ANSIBLE_VAULT_PASSWORD')
        //         ]) {
        //             sh '''
        //                 printf "%s" "$ANSIBLE_VAULT_PASSWORD" > vault-pass.txt

        //                 ansible-playbook \
        //                     -i ansible/inventory.ini \
        //                     ansible/playbook-2.yml \
        //                     --vault-password-file vault-pass.txt \
        //                     --extra-vars workspace="$WORKSPACE" \
        //                     --extra-vars docker_username="$DOCKER_USR" \
        //                     --extra-vars docker_password="$DOCKER_PSW"

        //                 rm -f vault-pass.txt
        //             '''
        //         }
        //     }
        // }

        stage('Train Model (CI)') {
            steps {
                echo 'Training and Selecting BEST Model...'
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                
                # 1. SETUP DVC & PULL DATA
                dvc remote add -d -f mylocal /tmp/dvc_store
                dvc pull
                
                # 2. LINK HISTORY & TRAIN
                ln -sf /var/lib/jenkins/mlflow_history mlruns
                python3 train.py
                
                # 3. THE OPTIMIZATION (Diet Plan)
                # Remove the link to the heavy history folder
                rm mlruns
                
                # Create a fresh, empty folder for the Docker image
                mkdir -p mlruns/latest_model
                
                # Find the LATEST 'model.pkl' file from the history
                # We sort by time (-t) and pick the first one (head -1)
                LATEST_MODEL=$(find /var/lib/jenkins/mlflow_history -name "model.pkl" -type f -printf "%T@ %p\n" | sort -n | tail -1 | cut -f2- -d" ")
                
                echo "Deploying Model: $LATEST_MODEL"
                
                # Copy ONLY that single file into the build folder
                # Your app.py is smart enough to find it anywhere in 'mlruns'
                cp "$LATEST_MODEL" mlruns/latest_model/
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
                echo 'Applying new K8s Config...'

                // 1. Apply Infrastructure (DB + Logging)
                sh "kubectl --kubeconfig=/var/lib/jenkins/kubeconfig apply -f k8s-database.yaml"
                sh "kubectl --kubeconfig=/var/lib/jenkins/kubeconfig apply -f k8s-logging.yaml"
                
                // 2. Apply Apps
                sh "kubectl --kubeconfig=/var/lib/jenkins/kubeconfig apply -f k8s-backend.yaml"
                sh "kubectl --kubeconfig=/var/lib/jenkins/kubeconfig apply -f k8s-frontend.yaml"

                // 3. Restart to pick up new images
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