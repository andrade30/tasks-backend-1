pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                bat 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment{
               scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=5f6fbb697c14719bc00b2a4f077e9e70ef554506 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java" 
                }
            }
        }
        stage ('Quality Gate'){
            steps {
                sleep(8)
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend'){
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('API Test'){
            steps {
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/andrade30/tasks-api-test'
                    bat 'mvn test'
                }    
            }
        }
        stage ('Deploy Frontend'){
            steps {
               dir('frontend') { 
                   git credentialsId: 'github_login', url: 'https://github.com/andrade30/tasks-frontend'
                   bat 'mvn clean package'
                   deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            } 
        }
        stage ('Functional Test'){
            steps {
                dir('functional-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/andrade30/tasks-functional-tests'
                    bat 'mvn test'
                }    
            }
        }
        stage('Deploy Prod'){
            steps{
                bat 'docker-compose build'
                bat 'docker-compose up -d'
            }
        }
        stage ('Health Check'){
            steps {
                sleep(8)
                dir('functional-test') {
                    bat 'mvn verify -Dskip.surefire.tests'
                }    
            }
        }
    }
    post{
        always{
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, functional-test/target/failsafe-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', followSymlinks: false, onlyIfSuccessful: true
        }
        unsuccessful{
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build $BUILD_NUMBER has failed', to: 'andrade.sp27@gmail.com'
        }
        fixed{
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build is fine!', to: 'andrade.sp27@gmail.com'
        }
    }
}

