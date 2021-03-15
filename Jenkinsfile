pipeline {
    agent {
        label 'slave'
    }

    tools {
        maven "mvn3"
    }
    
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }

    stages {
        
        stage('Clearing working app') {
            steps {
                sh "docker rm -f pandaapp || true"
            }
        }
        
        stage('GetCode') {
            steps {
                checkout scm
            }
        }
        
        stage('Build apllication && JUnit') {
            steps {
                sh "mvn clean install"
            }
        }

        stage('Docker image create') {
            steps {
                sh "mvn package -Pdocker -Dmaven.test.skip=true"
                //sh "mvn package -Pdockere"
            }
        }

        stage('Start application') {
            steps {
                sh "docker run -d --name pandaapp -p 0.0.0.0:8080:8080 -t ${IMAGE}:${VERSION}"
            }
        }
        
        stage('Selenium Test') {
            steps {
                sh "mvn test -Pselenium"
            }
        }
        
        stage('Deploy target to artifactory') {
            steps {
                configFileProvider([configFile(fileId: 'db3a0a26-7307-4a1b-876c-9292f55ac6ed', variable: 'MAVEN_SETTINGS')]) {
                //sh "mvn -s $MAVEN_SETTINGS deploy -Dmaven.test.skip=true"
                sh "mvn -s $MAVEN_SETTINGS deploy"
                }
            }
        } 
    }
    
    post {
        always {
            sh "docker stop pandaapp"
            junit '**/target/surefire-reports/TEST-*.xml'
            //archiveArtifacts 'target/*.jar'
            deleteDir()
        }
    }
}

