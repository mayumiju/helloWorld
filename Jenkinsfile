pipeline {
	agent any
	tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
    environment {
        dockerhub=credentials('dockerhub')
    }

	stages {
	    stage('Fetch code') {
            steps {
               git branch: 'master', url: 'https://github.com/mayumiju/helloWorld.git'
            }

	    }

	    stage('Build'){
	        steps{
	           sh 'mvn install -DskipTests'
	        }

	        post {
	           success {
	              echo 'Now Archiving it...'
	              archiveArtifacts artifacts: '**/target/*.jar'
	           }
	        }
	    }

	    stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis'){
            steps{
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis'){
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
                withSonarQubeEnv('sonar'){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=helloWorld \
                    -Dsonar.projectName=helloWorld \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate"){
            steps {
              timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                // Parameter indicates wheter to set pipeline to UNSTABLE if Quality Gates Fails
                // true = set pipeline to UNSTABLE , false = don't
                waitForQualityGate abortPipeline: true
                }
            }
	    }

	    stage ('Build image'){
	        steps {
	            sh 'docker build -t helloWorld-img:1.0 .'
	        }
	    }

	    stage ('Pushing to DockerHub'){
	        steps {
	            sh 'docker tag helloWorld-img:1.0 juliamayt/helloWorld:1.0 '
	            sh 'echo $dockerhub_PSW | docker login -u $dockerhub_juliamayt --password-stdin'

	            sh 'docker push juliamayt/helloWorld:1.0 '
	        }
	    }
	}
}