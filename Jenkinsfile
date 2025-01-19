pipeline {
    agent any

    tools {
        jdk 'JDK 17'
        maven 'maven 3.9.8'
    }

    environment {
        GIT_REPO = 'https://github.com/SubbuTechOps/spring-petclinic.git'
        GIT_BRANCH = 'develop'
        GIT_CREDENTIALS_ID = 'github-pat'

        SONARQUBE_HOST_URL = 'http://3.92.82.78:9000/'
        SONARQUBE_PROJECT_KEY = 'PetClinic'
        SONARQUBE_TOKEN = credentials('sonar-credentials')

        AWS_ACCOUNT_ID = '017820683847'
        ECR_REPO_URL = '017820683847.dkr.ecr.us-east-1.amazonaws.com/petclinic-dev'
        AWS_REGION = 'us-east-1'

        EKS_CLUSTER_NAME = 'demo-cluster'
        HELM_RELEASE_NAME = 'petclinic'
        MYSQL_RELEASE_NAME = 'mysql'
        HELM_NAMESPACE = 'petclinic-dev'
        MYSQL_NAMESPACE = 'mysql-dev'
        HELM_RELEASE_NAME_DB = 'mysql-release'

        TRIVY_DB_CACHE = "/var/lib/jenkins/trivy-db"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}", credentialsId: "${GIT_CREDENTIALS_ID}"
                stash name: 'source-code', includes: '**/*'
            }
        }

        stage('Trivy Scan Repository') {
            steps {
                sh """
                mkdir -p ${TRIVY_DB_CACHE}
                trivy fs --cache-dir ${TRIVY_DB_CACHE} --exit-code 1 --severity HIGH,CRITICAL .
                """
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test -DskipTests=false'
            }
        }

        stage('Generate JaCoCo Coverage Report') {
            steps {
                sh 'mvn jacoco:report'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                    -Dsonar.host.url=${SONARQUBE_HOST_URL} \
                    -Dsonar.login=${SONARQUBE_TOKEN}
                    """
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def COMMIT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.IMAGE_TAG = "${COMMIT_HASH}-${BUILD_NUMBER}"
                    env.DOCKER_IMAGE = "${ECR_REPO_URL}:${IMAGE_TAG}"

                    sh """
                    docker build -t ${DOCKER_IMAGE} .
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URL}
                    docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Trivy Scan Docker Image') {
            steps {
                sh """
                trivy image --exit-code 1 --severity HIGH,CRITICAL ${DOCKER_IMAGE}
                """
            }
        }
        
    stage('Install/Upgrade MySQL') {
    steps {
        script {
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access']]) {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                    
                    # Delete existing deployments
                    kubectl delete deployment mysql-db -n ${HELM_NAMESPACE} || true
                    kubectl delete deployment petclinic-app -n ${HELM_NAMESPACE} || true
                    
                    helm upgrade --install mysql-release ./petclinic-chart \
                        --namespace ${HELM_NAMESPACE} \
                        --create-namespace \
                        -f ./petclinic-chart/values.yaml \
                        --set app.enabled=false \
                        --set mysql.enabled=true
                """
            }
        }
    }
}

stage('Check MySQL Health') {
    steps {
        script {
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access']]) {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                    kubectl wait --namespace ${HELM_NAMESPACE} --for=condition=ready pod -l app=mysql --timeout=300s
                """
            }
        }
    }
}

stage('Deploy PetClinic Application') {
    steps {
        script {
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access']]) {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                     # Delete existing service
                    kubectl delete service petclinic-service -n ${HELM_NAMESPACE} || true
                    helm upgrade --install petclinic ./petclinic-chart \
                        --namespace ${HELM_NAMESPACE} \
                        -f ./petclinic-chart/values.yaml \
                        --set app.image.tag=${IMAGE_TAG} \
                        --set mysql.enabled=false
                """
            }
        }
    }
}
    
stage('Check PetClinic Health') {
    steps {
        script {
            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-access']]) {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER_NAME}
                    kubectl wait --namespace ${HELM_NAMESPACE} --for=condition=ready pod -l app=petclinic --timeout=300s
                """
            }
        }
    }
}
       
    }

    post {
        always {
            cleanWs()
            sh """
            docker image prune -f
            docker container prune -f
            """
        }
    }
}
