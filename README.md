pipeline {
    agent any

    environment {
        MAVEN_HOME = '/opt/maven'
        PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Clone Code') {
            steps {
                git credentialsId: 'github-creds', url: 'https://github.com/Vijay999712/java-hello-world-webapp.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        curl -v -u $TOMCAT_USER:$TOMCAT_PASS \
                        --upload-file target/java-hello-world.war \
                        "http://3.89.60.65:8080/manager/text/deploy?path=/myapp&update=true"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build and Deployment Successful!'
        }
        failure {
            echo 'Build or Deployment Failed!'
        }
    }
}
