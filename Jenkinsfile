pipeline{
    agent any
    stages{
        stage('Build BackEnd'){
            steps{
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests'){
            steps{
                sh 'mvn test'
            }
        }
        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps{
                withSonarQubeEnv('SONAR_LOCAL'){
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBackEnd -Dsonar.host.url=http://localhost:9000 -Dsonar.login=299ff3e749515552e395a9b2784205abc3d44d87 -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage('Quality Gate'){
            steps{
                sleep(5)
                timeout(time: 1, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy BackEnd'){
            steps{
                deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Test'){
            steps{
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/ophferreira/tasks-api-test'
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy FrontEnd'){
            steps{
                dir('frontend'){
                    git credentialsId: 'github_login', url: 'https://github.com/ophferreira/tasks-frontend'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat8(credentialsId: 'tomcat_login', path: '', url: 'http://localhost:8001')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }
        stage('Functional Test'){
            steps{
                dir('functional-test'){
                    git credentialsId: 'github_login', url: 'https://github.com/ophferreira/tasks-functional-test'
                    sh 'mvn test'
                }
            }
        }
        stage('Deploy Prod'){
            steps{
                sh 'echo G4sp4r1n4qazplm | sudo -S docker-compose build'
                sh 'echo G4sp4r1n4qazplm | sudo -S docker-compose up -d'
            }
        }
        stage('Health Check'){
            steps{
                sleep(5)
                dir('functional-test'){
                    sh 'mvn verify -Dskip.surefire.tests'
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
            emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build $BUILD_NUMBER has failed', to: 'pedrinho_360@hotmail.com'
        }
        fixed{
            emailext attachLog: true, body: 'See the attached log bellow', subject: 'Build is fine!!!', to: 'pedrinho_360@hotmail.com'
        }
    }
}