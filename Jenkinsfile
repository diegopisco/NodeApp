pipeline {
	agent any
	tools {
		nodejs 'NodeJS'
	}
	environment {
		DOCKER_HUB_CREDENTIALS_ID = 'jen-dockerhub'
		DOCKER_HUB_REPO = 'diegopisco/mirepo'
	}
	stages {
		stage('Checkout Github'){
			steps {
				git branch: 'main', credentialsId: 'c2f2fca6-b98a-4713-b966-5f29dbe7cbd2', url: 'https://github.com/diegopisco/NodeApp.git'
			}
		}		
		stage('Install node dependencies'){
			steps {
				sh 'npm install'
			}
		}
		/*stage('Test Code'){
			steps {
				sh 'npm test'
			}
		}*/
		stage('Build Docker Image'){
			steps {
				script {
					dockerImage = docker.build("${DOCKER_HUB_REPO}:latest")
				}
			}
		}
		/*stage('Trivy Scan'){
			steps {
				sh 'trivy --severity HIGH,CRITICAL --no-progress image --format table -o trivy-scan-report.txt ${DOCKER_HUB_REPO}:latest'
			}
		}*/
		stage('Push Image to DockerHub'){
			steps {
				script {
					docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}"){
						dockerImage.push('latest')
					}
				}
			}
		}
		stage('Install Kubectl'){
			steps {
				sh '''
				curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                		chmod +x kubectl
                		mv kubectl /usr/local/bin/kubectl
				'''
			}
		}
		stage('Deploy to Kubernetes'){
			steps {
				script {
					kubeconfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJWEoySDFDanh5YzB3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBME1UY3dOVEUzTlRsYUZ3MHpOVEEwTVRVd05URTNOVGxhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUURNbXRJTWdUM05TbVdITGNmaHZteGdIcTI4YXFCYjJqQVI3N0draHdsQXp4WHp2NmRkdkY2SHM2QWsKTUd2bnhMNFRjbTViZWtobFFleVdQandxNm1xK3pXWEYwcmNsek9jbmRtZHIxQVFqV2dlYkxzL1NDTHkwSUdBUApFaDljdGE0RnhuREprUjdmcWFBbDBVR2RlTDRVY0dQaVNHYnVyVnBDL2lhcUhZNGNuU0F1VFRTZGJGbkZKY2FKCnJqR1FGTGV5ejkrQjNudDdQTld1MXVKYlpjdXRYdXpvZHRuQkdxM0ZtZFZ0UExTTjJGNy9FMkU0VjA5VEJuaXMKUW9ZRUZ0S1Q3SmI2aEY4akQwbk5Sa1NSbTIxS1RLWUNGNE9Oc0gyWW1jS2tqSi9UWll5eUlIZE15cy9SOG93RAo1ZFRpVnpuZkFNR3FkSlg1d3Uxc2x2bjlkZy9sQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJUSzExNmt2aVhzQ3lSc1poMGNLQ1BkRzhKekxUQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQ2xVZlhJYXAycgpQY0pJR3hYU1ZXMWpaV2wxN0hZV3c4cXl1NGtPVkpsa1VLZU82UnRxa2FrVjJpQVhmdlI2SExpMlF2YmJDUzZFCjFhdnhzN2tXckRybEZYOWNIQlNaWjcyVzExZllPY2NoZ3FSbmFuUkNJSUc1Mk11ZXNSdUY5MlRaR0xmQWI4Z3AKcnlhQWZ3dGs5MG03ZTMyd3lVL1dDQWJrZzNENStSM3Z2NElWOXJaUTQrVkN2dXN4UUZZd2FaTG9PYmkzajFXbwp4aEpZZmRwSEtzTkhNQWQ0czBPVDQwejRXQjc2RXpXd0N5d2VpVVM1TGJVT0JLeWdFQXU5czBIdGtaVFNQam12CjZwNmhrTHdPTDlzRHBQa1QzdWt4cklGTHZZOFlDS0p1Nlp4aXVHUG1ORDhEY1FTT3d5U29vN1VYUDRPd3hvdHgKeXZIL3pRNWtYTVJYCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K', credentialsId: 'kubeconfig', serverUrl: 'https://kubernetes.docker.internal:6443') {
    						sh 'kubectl apply -f deployment.yaml'
					}
				}
			}
		}
	}

	post {
		success {
			echo 'Build&Deploy completed succesfully!'
		}
		failure {
			echo 'Build&Deploy failed. Check logs.'
		}
	}
}
