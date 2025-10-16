@Library('Shared') _

pipeline {
    agent any
    
    environment {
        // Updated to use your Docker Hub username
        DOCKER_IMAGE_NAME = 'ankitkothalkar/easyshop-app'
        DOCKER_MIGRATION_IMAGE_NAME = 'ankitkothalkar/easyshop-migration'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        // Using a single variable for Docker credentials
        DOCKER_CREDENTIALS = 'docker-hub-credentials'
        
        // --- FIX: Credentials function removed, passing only ID as string ---
        GITHUB_CREDENTIALS = 'github-credentials' 
        
        GIT_BRANCH = "master"
    }
    
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    clean_ws()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                script {
                    // Update this URL if your repository is different
                    clone("https://github.com/ankitkothalkar/tws-e-commerce-app.git","master")
                }
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build Main App Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'Dockerfile',
                                context: '.'
                            )
                        }
                    }
                }
                
                stage('Build Migration Image') {
                    steps {
                        script {
                            docker_build(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                dockerfile: 'scripts/Dockerfile.migration',
                                context: '.'
                            )
                        }
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                script {
                    run_tests()
                }
            }
        }
        
        stage('Security Scan with Trivy') {
            steps {
                script {
                    trivy_scan()
                }
            }
        }
        
        stage('Push Docker Images') {
            parallel {
                stage('Push Main App Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: env.DOCKER_CREDENTIALS
                            )
                        }
                    }
                }
                
                stage('Push Migration Image') {
                    steps {
                        script {
                            docker_push(
                                imageName: env.DOCKER_MIGRATION_IMAGE_NAME,
                                imageTag: env.DOCKER_IMAGE_TAG,
                                credentials: env.DOCKER_CREDENTIALS
                            )
                        }
                    }
                }
            }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    update_k8s_manifests(
                        imageTag: env.DOCKER_IMAGE_TAG,
                        manifestsPath: 'kubernetes',
                        // Use the fixed GITHUB_CREDENTIALS string ID
                        gitCredentials: env.GITHUB_CREDENTIALS, 
                        gitUserName: 'Jenkins CI',
                        gitUserEmail: 'reportankitk@gmail.com'
                    )
                }
            }
        }
        
        // --- NEW STAGE: Deployment to EKS ---
        stage('Deployment to EKS') {
            steps {
                script {
                    // Apply all YAML files in the kubernetes directory
                    sh "kubectl apply -f kubernetes/"
                    echo "Application deployed to EKS cluster successfully."
                }
            }
        }
    }
}
