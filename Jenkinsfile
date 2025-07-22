pipeline {
    agent any

    environment {
        MAVEN_HOME = '/usr/share/maven'
        JAVA_HOME = '/usr/lib/jvm/java-11-openjdk'
        DEPLOY_SERVER = 'http://localhost:8080/manager/text'
        DEPLOY_USER = 'admin'
        DEPLOY_PASS = 'admin_password'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Baffoe6/maven-webapp.git'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Code Review') {
            steps {
                sh 'mvn checkstyle:check'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def warName = sh(script: "ls target/*.war", returnStdout: true).trim()
                    withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                        sh """
                            curl --upload-file ${warName} \
                            --user ${TOMCAT_USER}:${TOMCAT_PASS} \
                            ${DEPLOY_SERVER}/deploy?path=/maven-webapp&update=true
                        """
                    }
                }
            }
        }
    }
}