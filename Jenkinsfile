pipeline {
    agent any
    stages {
        stage('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=b1bf3754ea81a0abdfd52e3d141c5d81f81105a6 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage('Quality Gate'){
            steps {
                sleep(40)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy Backend'){
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Test'){
            steps {
                dir('api-test') {
                    git credentialsId: 'GitLogin', url: 'https://github.com/mruthes/tasks-api-test.git'
                    bat 'mvn test'
                }
            }
        }
        stage('Deploy Frontend'){
            steps {
                dir('frontend') {
                    git credentialsId: 'GitLogin', url: 'https://github.com/mruthes/tasks-frontend'
                    bat 'mvn clean package -DskipTests=true'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
               
            }
        }
        stage('Functional Test'){
            steps {
                dir('functional-test') {
                    git credentialsId: 'GitLogin', url: 'https://github.com/mruthes/tasks-functional-test.git'
                    bat 'mvn test'
                }
            }
        }
        stage("Deploy Prod"){
            steps{
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
        stage('Health Check'){
            steps {
                sleep(20)
                dir('functional-test') {
                    bat 'mvn verify -Dskip.surefire.tests'
                }
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-tests/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war', 'frontend/target/tasks.war', onlySuccessful: true
        }
        unsuccessul {
            emailext attachlog: true, body: 'See the attached log below',subject : 'Build $BUILDER_NUMBER has failed'
        }
        fixed {
            emailext attachlog: true, body: 'See the attached log below',subject : 'Build his fine!!!',to: 
        }
    }
}
