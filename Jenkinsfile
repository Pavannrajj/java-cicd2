pipeline {
    agent any

    tools {
        maven 'maven_m3'
        ansible 'ansible'
    }

    stages {
        stage('Hello') {
            steps {
                git branch: 'main', url: 'https://github.com/Siddeshg672/hello_world_public_war.git'
            }
        }

        stage('sonar_scan') {
            steps {
                script {
                    withSonarQubeEnv('sonar-hiqode') {
                        sh "mvn clean verify sonar:sonar -Dsonar.projectKey=sonar-scan-with-jenkins -Dsonar.projectName='sonar-scan-with-jenkins'"
                    }
                }
            }
        }

        stage('build binaries') {
            steps {
                sh 'mvn clean install'
                sh 'cp target/java-frontend-app.war .'
                sh 'mv java-frontend-app.war java-frontend-app-${BUILD_NUMBER}.war'
            }
        }

        stage('push binaries to nexus') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-cred', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh """
                        curl -u${user}:${pass} -T java-frontend-app-${BUILD_NUMBER}.war \"https://your-nexus-host/repository/your-nexus-repo/java-frontend-app-${BUILD_NUMBER}.war\"
                        """
                    }
                }
            }
        }

        stage('deploy to tomcat') {
            steps {
                script {
                    sh """
                    chmod 400 /var/lib/jenkins/demo-aws.pem
		    ansible-playbook deploy_tomcat.yml -i hosts.ini --private-key /var/lib/jenkins/demo-aws.pem -u ubuntu -e 'BUILD_NUMBER=${BUILD_NUMBER}'
		    """
                }
            }
        }
    }
}
