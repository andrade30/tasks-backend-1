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
                    bat "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=5f6fbb697c14719bc00b2a4f077e9e70ef554506 -Dsonar.language=java -Dsonar.java.binaries=target/classes  
                }
            }
        }
        stage ('Quality Gate'){
            steps {
                sleep(5)
                timeout(time: 2, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('Deploy Backend'){
            steps {
                deploy adapters: [tomcat8(credentialsId: 'github_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
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
                   deploy adapters: [tomcat8(credentialsId: 'github_login', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
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
    }
}

