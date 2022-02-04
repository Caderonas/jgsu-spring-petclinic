// Install sonarqube docker-pipepline docker
pipeline {
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
