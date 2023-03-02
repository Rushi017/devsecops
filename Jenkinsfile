pipeline{
    agent any
    environment{
imageName = "nava9594/$JOB_NAME:v1.$BUILD_ID"
}
    triggers {
        pollSCM 'H/2 * * * *'
		}
    tools{
        maven 'maven'
    }
    stages{
        stage('checkout the code'){
            steps{
                slackSend channel: 'jenkins-notification', message: 'job started'
                git url:'https://github.com/Rushi017/devsecops', branch: 'main'
            }
        }
        stage('build the code'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('Mutation Tests - PIT') {
          steps {
           sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
       }
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(installationName: 'sonar-scanner', credentialsId: 'jenkins-sonar-token') {
                            sh "mvn sonar:sonar -f /var/lib/jenkins/workspace/hello-world-cicd/pom.xml"
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                } 
            }
        }
        stage('create docker image'){
            steps{
                sh '''docker image build -t $JOB_NAME:v1.$BUILD_ID .
docker image tag $JOB_NAME:v1.$BUILD_ID nava9594/$JOB_NAME:v1.$BUILD_ID
docker image tag $JOB_NAME:v1.$BUILD_ID nava9594/$JOB_NAME:latest'''
            }

        }
        stage('push the image into docker hub'){
            steps{
                withCredentials([string(credentialsId: 'Docker pass', variable: 'docker_pass')]) {
                    sh "docker login -u nava9594 -p ${docker_pass}"

}
                    sh '''docker image push nava9594/$JOB_NAME:v1.$BUILD_ID
docker image push nava9594/$JOB_NAME:latest 
docker image rmi $JOB_NAME:v1.$BUILD_ID nava9594/$JOB_NAME:v1.$BUILD_ID nava9594/$JOB_NAME:latest'''

            }
        }
        stage('Deploy application on k8s'){
            steps{
                sh 'sed -i "s#replace#${imageName}#g" deployment.yaml'
                sh 'kubectl apply -f deployment.yaml'
        }
        }
    }
        post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
            slackSend channel: 'jenkins-notification', message: ' job success'
        }
        failure{
            echo "========pipeline execution failed========"
            slackSend channel: 'jenkins-notification', message: ' job failed'
        }
        }
    }
