// Install sonarqube docker-pipepline docker
pipeline {

	agent {
		node {
			label 'docker'
		}
	}

	options {
		timestamps()
	}

	environment {
		IMAGE = readMavenPom().getArtifactId()
		VERSION = readMavenPom().getVersion()
	}

	stages {
		stage('Build') {
			agent {
				docker {
					reuseNode true
					image 'maven:3.5.0-jdk-8'
				}
			}
			steps {
				withMaven(options: [findbugsPublisher(), junitPublisher(ignoreAttachments: false)]) {
					sh 'mvn -T8C dependency:go-offline'
					sh 'mvn -T8C spring-javaformat:help'
					sh 'mvn -T8C spring-javaformat:apply'
					sh 'mvn -T8C clean findbugs:findbugs package'
				}
			}
			post {
				success {
					archiveArtifacts(artifacts: '**/target/*.jar', allowEmptyArchive: true)
				}
			}
		}
		stage('Quality Analysis') {
			parallel {
				stage ('Integration Test') {
					agent any
					steps {
						echo 'Run integration tests here...'
						echo 'mvn -T8C test'
					}
				}
				node {
					stage('SCM') {
						checkout scm
					}
					stage('SonarQube Analysis') {
						def mvn = tool 'Default Maven';
						withSonarQubeEnv() {
							sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=Caderonas_spring-petclinic_AX6_8LHDV0s42Ye3Pgw3"
						}
					}
				}
			}

		stage('Build and Publish Image') {
			when {
				branch 'main'  //only run these steps on the master branch
			}
			steps {
				/*
				* Multiline strings can be used for larger scripts. It is also possible to put scripts in your shared library
				* and load them with 'libaryResource'
				*/
				sh """
				docker build -t ${IMAGE} .
				docker tag ${IMAGE} ${IMAGE}:${VERSION}
				docker push ${IMAGE}:${VERSION}
				"""
			}
		}
	}

	post {
		failure {
			// notify users when the Pipeline fails
			mail to: 'team@example.com',
			subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
			body: "Something is wrong with ${env.BUILD_URL}"
		}
	}
}
