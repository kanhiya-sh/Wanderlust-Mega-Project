@Library('Shared') _
pipeline {
    agent {label 'Node'}
    
    environment{
        SONAR_HOME = tool "Sonar"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: '', description: 'Setting docker image for latest push')
    }
    
    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.FRONTEND_DOCKER_TAG == '' || params.BACKEND_DOCKER_TAG == '') {
                        error("FRONTEND_DOCKER_TAG and BACKEND_DOCKER_TAG must be provided.")
                    }
                }
            }
        }
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
        stage('Git: Code Checkout') {
            steps {
                script{
                    code_checkout("https://github.com/kanhiya-sh/Wanderlust-Mega-Project.git","main")
                }
            }
        }
        
        stage("Trivy: Filesystem scan"){
            steps{
                script{
                    trivy_scan()
                }
            }
        }

        stage("OWASP: Dependency check"){
            steps{
                script{
                    owasp_dependency()
                }
            }
        }
        
        stage("SonarQube: Code Analysis"){
            steps{
                script{
                    sonarqube_analysis("Sonar","wanderlust","wanderlust")
                }
            }
        }
        stage('Exporting environment variables') {
            parallel{
                stage("Backend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatebackendnew.sh"
                            }
                        }
                    }
                }
                
                stage("Frontend env setup"){
                    steps {
                        script{
                            dir("Automations"){
                                sh "bash updatefrontendnew.sh"
                            }
                        }
                    }
                }
            }
        }
        
        stage("Docker: Build Images"){
            steps{
                script{
                        dir('backend'){
                            docker_build("wanderlust-backend-beta","${params.BACKEND_DOCKER_TAG}","kanhiyasharma18")
                        }
                    
                        dir('frontend'){
                            docker_build("wanderlust-frontend-beta","${params.FRONTEND_DOCKER_TAG}","kanhiyasharma18")
                        }
                }
            }
        }
        
        stage("Docker: Push to DockerHub"){
            steps{
                script{
                    docker_push("wanderlust-backend-beta","${params.BACKEND_DOCKER_TAG}","kanhiyasharma18") 
                    docker_push("wanderlust-frontend-beta","${params.FRONTEND_DOCKER_TAG}","kanhiyasharma18")
                }
            }
        }
        stage("Deploy to EC2") {
    steps {
        script {
            sh """
                echo "Pulling latest backend image..."
                docker pull kanhiyasharma18/wanderlust-backend-beta:${params.BACKEND_DOCKER_TAG} || exit 1

                echo "Pulling latest frontend image..."
                docker pull kanhiyasharma18/wanderlust-frontend-beta:${params.FRONTEND_DOCKER_TAG} || exit 1

                echo "Stopping old containers if running..."
                docker rm -f wanderlust-backend || true
                docker rm -f wanderlust-frontend || true

                echo "Starting new backend container..."
                docker run -d --name wanderlust-backend -p 5000:5000 kanhiyasharma18/wanderlust-backend-beta:${params.BACKEND_DOCKER_TAG}

                echo "Starting new frontend container..."
                docker run -d --name wanderlust-frontend -p 80:80 kanhiyasharma18/wanderlust-frontend-beta:${params.FRONTEND_DOCKER_TAG}

                echo "Currently running containers:"
                docker ps
            """
        }
    }
}

    }
    post{
        success{
            archiveArtifacts artifacts: '*.xml', followSymlinks: false
        }
    }
}
