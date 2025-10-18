pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'aminata286'
        KUBECONFIG = '/var/lib/jenkins2/.kube/config' // chemin vers ton kubeconfig sur le serveur Jenkins
    }

  triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'ref', value: '$.ref'],
                [key: 'before', value: '$.before']
            ],
            causeString: 'Triggered on $ref',
            token: 'mysecret',
            printContributedVariables: true,
            printPostContent: true
        )
    }
    stages {

        // 🧩 Étape 1 : Récupération du code source
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Aminata11/kubernetes.git'
            }
        }

        

        // 🔑 Étape 4 : Connexion à Docker Hub
        stage('Login to DockerHub') {
            steps {
                echo 'Connexion à Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'jen-kubernetes', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        // 🛠️ Étape 5 : Construction de l’image backend
        stage('Build Backend Image') {
            steps {
                echo 'Construction de l’image backend...'
                sh 'docker build -t $DOCKER_HUB_REPO/backend:latest ./mon-projet-express'
            }
        }

        // 🛠️ Étape 6 : Construction de l’image frontend
        stage('Build Frontend Image') {
            steps {
                echo 'Construction de l’image frontend...'
                sh 'docker build -t $DOCKER_HUB_REPO/frontend:latest ./'
            }
        }

        // 📤 Étape 7 : Push des images vers Docker Hub
        stage('Push Images') {
            steps {
                echo 'Envoi des images vers Docker Hub...'
                sh '''
                    docker push $DOCKER_HUB_REPO/backend:latest
                    docker push $DOCKER_HUB_REPO/frontend:latest
                '''
            }
        }

        // 🚀 Étape 8 : Déploiement sur Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                echo "Déploiement sur le cluster Kubernetes..."
               sh '''
                  kubectl apply -f K8s/mongo-deployment.yaml
                  kubectl apply -f K8s/backend-deployment.yaml
                  kubectl apply -f K8s/frontend-deployment.yaml
                  '''

            }
        }
    }

    // 📬 Étapes post-pipeline
    post {
        success {
            emailext(
                subject: "✅ Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline réussi 🎉\nDétails : ${env.BUILD_URL}",
                to: "seckaminata87@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "❌ Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Le pipeline a échoué 😞\nDétails : ${env.BUILD_URL}",
                to: "seckaminata87@gmail.com"
            )
        }
        always {
            echo 'Nettoyage des images et conteneurs Docker...'
            sh '''
                docker container prune -f
                docker image prune -f
                docker logout
            '''
        }
    }
}
