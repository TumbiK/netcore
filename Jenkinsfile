pipeline {
    environment {
        DEPLOY = "${env.BRANCH_NAME == "main" || env.BRANCH_NAME == "develop" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "main" ? "develop" : "example-staging"}"
        VERSION = '1.0'
        DOMAIN = 'localhost'
        REGISTRY = 'k3d-erpregistry.localhost:5000/core'
        REGISTRY_CREDENTIAL = ''
    }
    agent any 
        
    
    stages {
        stage('Build') {
		agent {
				label 'windows_slave'
			}
            steps {                
                    'dotnet --version'
				   }
        }
        stage('Docker Build') {
		agent {
			kubernetes {
            
            yamlFile 'build.yaml'
			}
			}
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    sh "docker build -t ${REGISTRY}:${VERSION} ."
                }
            }
        }
        stage('Docker Publish') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    withDockerRegistry([credentialsId: "${REGISTRY_CREDENTIAL}", url: ""]) {
                        sh "docker push ${REGISTRY}:${VERSION}"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('helm') {
                    sh "helm upgrade --install --force --set name=${NAME} --set image.tag=${VERSION} --set domain=${DOMAIN} ${NAME} ./helm"
                }
            }
        }
    }
}