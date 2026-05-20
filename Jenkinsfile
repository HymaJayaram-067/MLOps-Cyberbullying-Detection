pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "hymajayaram067/cyberbullying-app"
    }

    stages {
        stage('Setup Permissions') {
            steps {
                script {
                    sh '''
                    echo "Granting permissions to the Jenkins user.."
                    sudo usermod -aG docker jenkins
                    sudo mkdir -p /var/lib/jenkins/.ssh
                    sudo chown -R jenkins:jenkins /var/lib/jenkins/.ssh
                    sudo chmod 700 /var/lib/jenkins/.ssh
                    '''
                }
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/HymaJayaram-067/MLOps-Cyberbullying-Detection.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'ls -l'
                sh 'sudo docker build -t ${DOCKER_IMAGE} .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'docker-credentials-id',
                                     usernameVariable: 'DOCKER_USERNAME',
                                     passwordVariable: 'DOCKER_PASSWORD')
                ]) {
                    sh '''
                    echo "Logging in to Docker Hub..."
                    echo "${DOCKER_PASSWORD}" | sudo docker login -u "${DOCKER_USERNAME}" --password-stdin
                    sudo docker push ${DOCKER_IMAGE}
                    '''
                }
            }
        }

	 stage('Run Ansible Deployment') {
            steps {

		withCredentials([string(credentialsId: '1f19b1cc-84d4-48e6-97dd-78c831657141', variable: 'ANSIBLE_PASS')]){ 
                sh '''
                    echo "Creating Ansible inventory file..."
                    echo "[myhosts]" > inventory.ini
                    echo "localhost ansible_connection=local ansible_become_pass=${ANSIBLE_PASS}" >> inventory.ini

                    echo "Running Ansible Playbook..."
		    ansible-playbook -i inventory.ini deploy_webapp.yml --extra-vars "ansible_become_pass=${ANSIBLE_PASS}" -vvvv
                '''
		}
            }
        }

	stage('Start Minikube and Apply Kubernetes Resources') {
    	    steps {
                script {
            echo "Cleaning up old Minikube instances..."
            sh '''
            if docker ps -a --format '{{.Names}}' | grep -q "^minikube$"; then
                echo "Old minikube container exists. Deleting..."
                minikube delete || true
                docker rm -f minikube || true
            fi
            '''

            echo "Starting Minikube..."
            sh 'minikube start --driver=docker'

            echo "Applying Kubernetes resources..."
            sh 'kubectl apply -f kubernetes/'
           }
        }
     }


        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying webapp to Kubernetes..."
                    sh 'kubectl set image deployment/webapp-deployment webapp-container=${DOCKER_IMAGE}'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    echo "Verifying the Kubernetes deployments..."
                    sh 'kubectl get pods'
                    sh 'kubectl get services'
                }
            }
        }
    }

    post {
        success {
            echo 'Build and push succeeded!'
        }
        failure {
            echo 'Build or push failed!'
        }
    }
}

