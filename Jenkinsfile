pipeline {
    
	agent any

    environment {
        resgistry = "csag095/vprofileapp"
        resgistryCredentials = "dockerhub"
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

        stage('Build App Image'){
            steps{
                script{
                   dockerImage = docker.build resgistry + "v:$BUILD_NUMBER"
                }
            }
        }

        stage('Push Image to DockerHub'){
            steps{
                script{
                    docker.withRegistry('', resgistryCredentials) {
                    dockerImage.push("v$BUILD_NUMBER")
                    dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Remove Unused docker image') {
            steps{
                sh "docker rmi $registry:v$BUILD_NUMBER"
            }
        }

        stage('Deploy on kubernetes with helm'){
            agent{label 'KOPS'}
            steps{
                sh 'helm upgrade --install --force vpro-application-stack helm/vprocharts --set appimage=${registry}:${BUILD_NUMBER} --namespace prod'
            }
        }


    }


}
