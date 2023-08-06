pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000/ -Dsonar.login=2e77aa03d0c63ade223d6000c94e99a9ee130316 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage ('Quality Gate') {
            steps {
                sleep(60)
                timeout(time: 30, unit: 'SECONDS') {
                    waitForQualityGate abortPipeline: true, credentialsId: 'SonarToken'
                }
            }
        }
        stage ('Deploy Tomcat') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage ('Tests API') {
            steps {
                dir('api-test') {
                    git 'git@github.com:Pedrodkp/jenkins-tasks-api-test.git'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend AllSteps') {
            steps {
                dir('frontend') {
                    git 'git@github.com:Pedrodkp/jenkins-tasks-frontend.git'
                    sh 'mvn clean package -DskipTests=true'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage ('Functional Tests') {
            steps {
                dir('functional-test') {
                    git 'git@github.com:Pedrodkp/jenkins-tasks-functional-test.git'
                    sh 'mvn test'
                }
            }
        }
        stage ('Health Check') {
            steps {
                sh 'curl -s -o /dev/null -w "%{http_code}" http://localhost:8001/tasks/'
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, functional-test/target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', onlyIfSuccessful: true
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log', subject: 'Build $BUILD_NUMBER has failed', to: 'pedro.pereira@upiara.com, contato@upiara.dev.br'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log', subject: 'Build is fine', to: 'pedro.pereira@upiara.com, contato@upiara.dev.br'
        }
    }
}