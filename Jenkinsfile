pipeline {
    agent none
    tools {
        maven 'maven install' 
    }
    parameters{
        string(name: 'Env', defaultValue: 'Dev', description: 'in which env you wnat to to build')
        booleanParam(name: 'unit-test', defaultValue: true, description: 'processed with unit test or not')
        choice(name: 'package-version', choices: ['1.1 version', '1.2 version', '1.3 version'], description: 'package version need to select')
    } 
     environment {
    BUILD_SERVER="ec2-user@172.31.17.122"
    }

    stages {
        stage('compile') {
            agent any
            steps {
                script{
                sshagent(['slave-3']) {
                echo "you are compiling code in ${params.Env}"
                echo 'compiling the code '
                sh 'mvn clean compile'
                sh "scp -o StrictHostkeychecking=no server-script.sh ${BUILD_SERVER}:/home/ec2-user/server-script.sh"
                sh "ssh -o StrictHostkeychecking=no ${BUILD_SERVER} 'bash /home/ec2-user/server-script.sh'"
            }   
            }
        }
    }
        stage('code review') {
           agent {label 'slave-1'}
            steps {
                script{
                echo 'code review using pmd plugin '
                sh 'mvn pmd:pmd'
            }
            }
        }
        stage('unit test') {
            agent {label 'slave-1'}
            when {
                expression {
                    params.unit-test == true
                }
            }
            steps {
                script{
                echo 'code revietest using junit plugin'
                echo "you want to skip testing ? ${params.unit-test}"
                sh 'mvn test'
            }
            }
        }
          stage('code analysis') {
           agent {label 'slave-1'}
            steps {
                script{
                echo 'code analysis using jacoco plugin'
                sh 'mvn verify'
            }
            }
        }
        stage('code package') {
            agent {label 'slave-1'}
            steps {
                script{
                echo 'code package using maven'
                echo "code package version is ${params.package-version}"
                sh 'mvn package'
                }
            }
        }
        stage('code deploy') {
            agent {label 'slave-1'}
            steps {
            script{
                echo 'artifact upload to jfrog'
                sh 'mvn -U deploy -s settings.xml'
                }
            }
        }
    }
}
