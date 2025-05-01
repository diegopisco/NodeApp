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
					kubeconfig(caCertificate: 'LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURRakNDQWlxZ0F3SUJBZ0lJV1BqNDFSTVhVS2d3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TlRBME1UY3dOVEUzTlRsYUZ3MHlOakEwTVRjd05URTNOVGxhTURZeApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sc3dHUVlEVlFRREV4SmtiMk5yWlhJdFptOXlMV1JsCmMydDBiM0F3Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRREtlc05BR3A2QyttZ2oKSFBjOE9EWDZuTllnUnh6UnMrTlByYkNoUzBETDBDMjk1SjlUUjl1WkZiUUdsd2YxZDNzQ2lDQU1NWE5kVWVNOApMaytDZ0VqTVJvMStBb0dsL2MxR08xWjRmaHY3cEhCdlQzcWF5UDIydXZadFNWNjZjWm5oR015SnY0S0p1UWRRCjc0WWR0aDZFc2hSZW15Q09QcTFGeXRXbW5BVCtXaWZOZExDM3k2UTVYd1I2eDdYcHlyOGp0SG5RZS9WTUo1T1oKOFBtS2JqMnRsSVBhanBSMWR3cjd2Y1RIdEFVRGZGTGh6bmFLcUg2SnB5N1grY1BKYnF1Uk5leGdrK08wTldsZwpKeHNCTWE5c0YzZEpwYktTdUwydlpYbm52M1RoSkxXTVNDNlVwZ0hWR2hsYnJGWXphSC9ubWtwd3BiQSt3Q0gyCkpsZ0tSZlNoQWdNQkFBR2pkVEJ6TUE0R0ExVWREd0VCL3dRRUF3SUZvREFUQmdOVkhTVUVEREFLQmdnckJnRUYKQlFjREFqQU1CZ05WSFJNQkFmOEVBakFBTUI4R0ExVWRJd1FZTUJhQUZNclhYcVMrSmV3TEpHeG1IUndvSTkwYgp3bk10TUIwR0ExVWRFUVFXTUJTQ0VtUnZZMnRsY2kxbWIzSXRaR1Z6YTNSdmNEQU5CZ2txaGtpRzl3MEJBUXNGCkFBT0NBUUVBS1dTSU9LcUZaWStkem5MR1drZDFyVFE5dEcrcWRjNHBIVWQzaXZOS1lld1Zic3N5cjJpa1g5SHkKaVNORVhhb3l2TDdkSVhONFBRenh3M1FTd0ZwTTlCVWVRRnhYaGp2dzZTb1ZFRUw1QkhnT09YdmxOc0lWUktJZQpLbHh4QlV0TTJMSU4wQmw1dnZpV0JJNUNhT1QrUmE5dXJRdzFVajJTdGFsakRhT1dzeG51NU5TYVFxdTdudEc5CmpDeVN2RWVyS2VSWG1TZHJPN3BRcVprRG9BRWJLNkZ6cnJiZEk2R1FCU0J1c2FQN0doMXBoRHphT2taMlcrRlEKV0J0SjFiSWRPWHIyalBYK2pFUGl2eld0T2lyay8wL2NjcStMMmRYanlOb09leTdZTFVvengxaGtGbUNLUlNKVAoyYWNFZ0ZWU3ZHQTdScXZBVXF0bUV0Y2VVSm9Od1E9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==', credentialsId: 'kubeconfig', serverUrl: 'https://kubernetes.docker.internal:6443') {
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
