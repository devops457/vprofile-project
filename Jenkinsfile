pipeline {
    
	agent any
/*	
	tools {
        maven "maven3.9.5"
    }
*/	
    environment {
        registry = "ayushs60486/vproappdock"
	registryCredentials = 'dockerhub'
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

	stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

	stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonarscanner4'
          }

          steps {
            withSonarQubeEnv('sonar-pro') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }

            timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
            }
          }
        }

        stage('Build App Image') {
		steps {
			script {
				dockerImage = docker.build registry + ":V$BUILD_NUMBER"
			}
		}
	}
        
        stage('Upload Image') {
		steps {
			script {
				docker.withRegistry ('', registryCredentials) {
					dockerImage.push("V$BUILD_NUMBER")
					dockerImage.push('latest')
				}			
			}
		}
	}

	stage ('Remove unused Docker Images') {
		steps {
			sh "docker rmi $registry:V$BUILD_NUMBER"
		}
	}						 


	stage ('Kubernetes Deploy') {
		agent {label 'KOPS'}
		steps {
			sh "helm upgrade --install --force vprofile-stack helm/vprofilelcharts --set appimage=${registry}:V${BUILD_NUMBER}  --namespace prod"
		}
	}						 
    }


}
