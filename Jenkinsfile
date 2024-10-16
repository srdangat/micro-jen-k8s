pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
        HOMEPAGE_REPO = "homepage"
        LAPTOPPAGE_REPO = "laptoppage"
        MOBILEPAGE_REPO = "mobilepage"
        HOMEPAGE_TAG = "latest"
        LAPTOPPAGE_TAG = "latest"
        MOBILEPAGE_TAG = "latest"
        HOMEPAGE_REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${HOMEPAGE_REPO}"
        LAPTOPPAGE_REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${LAPTOPPAGE_REPO}"
        MOBILEPAGE_REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${MOBILEPAGE_REPO}"
        CLUSTER_NAME = "eksdemo1"
    }

    stages {
        stage('Login to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds') {
                        sh """
                            aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${HOMEPAGE_REPOSITORY_URI} &&
                            aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${LAPTOPPAGE_REPOSITORY_URI} &&
                            aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${MOBILEPAGE_REPOSITORY_URI}
                        """
                    }
                }
            }
        }
        stage('Git Checkout') {
            steps {
                script {
                    checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/srdangat/micro-jen-k8s.git']])
                }
            }
        }
        stage('Build Docker Images') {
            steps {
                sh """
                    docker build -t ${HOMEPAGE_REPOSITORY_URI}:${HOMEPAGE_TAG} ./home
                    docker build -t ${LAPTOPPAGE_REPOSITORY_URI}:${LAPTOPPAGE_TAG} ./laptop
                    docker build -t ${MOBILEPAGE_REPOSITORY_URI}:${MOBILEPAGE_TAG} ./mobile
                """
            }
        }
        stage('Tag Docker Images') {
            steps {
                script {
                    sh """
                        docker tag ${HOMEPAGE_REPOSITORY_URI}:${HOMEPAGE_TAG} ${HOMEPAGE_REPOSITORY_URI}:${HOMEPAGE_TAG}
                        docker tag ${LAPTOPPAGE_REPOSITORY_URI}:${LAPTOPPAGE_TAG} ${LAPTOPPAGE_REPOSITORY_URI}:${LAPTOPPAGE_TAG}
                        docker tag ${MOBILEPAGE_REPOSITORY_URI}:${MOBILEPAGE_TAG} ${MOBILEPAGE_REPOSITORY_URI}:${MOBILEPAGE_TAG}
                    """
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    sh """
                        docker push ${HOMEPAGE_REPOSITORY_URI}:${HOMEPAGE_TAG} &&
                        docker push ${LAPTOPPAGE_REPOSITORY_URI}:${LAPTOPPAGE_TAG} &&
                        docker push ${MOBILEPAGE_REPOSITORY_URI}:${MOBILEPAGE_TAG}
                    """                
                }
            }
        }
        stage('Clean Up Docker Images') {
            steps {
                script {
                    sh """
                        docker system prune -af
                    """
                }
            }
        }
        stage('Configure kubectl') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds') {
                        sh """
                            aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${CLUSTER_NAME}
                        """
                    }
                }
            }
        }
        stage('Get Nodes') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds') {
                        sh """
                            kubectl get nodes
                        """
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds') {
                        sh """
                            kubectl apply -f home/home.yaml &&
                            kubectl apply -f laptop/laptop.yaml &&
                            kubectl apply -f mobile/mobile.yaml &&
                            kubectl apply -f ingress/ingress.yaml
                        """
                    }
                }
            }
        }
    }
}