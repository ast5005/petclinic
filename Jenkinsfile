pipeline {
    agent any
       triggers {
        pollSCM "*/5 * * * *"
       }
    stages {
        stage('Build Application') { 
            steps {
                echo '=== Building Petclinic Application ==='
                sh 'mvn -B -DskipTests clean package' 
            }
        }
        stage('Test Application') {
            steps {
                echo '=== Testing Petclinic Application ==='
                sh 'mvn test'
              
        
            // Run OWASP Dependency Check
            dependencyCheck additionalArguments: '--out dependency-report.xml --format XML', odcInstallation: '5.0.0'
            dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            
            slackSend botUser: true, channel: 'security', color: '#ff0000', message: '21 Critical 17 High Vulnerability Detected ', teamDomain: 'futurama', tokenCredentialId: 'slack-token', username: 'jenkins'        
             slackUploadFile channel: 'security', credentialId: 'slack-token', filePath: 'dependency-report.xml', initialComment: 'Dependency Check Report'
    }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Building Petclinic Docker Image ==='
                script {
                    echo '===Inside Script==='
                    app = docker.build("0327201/petclinic-spinnaker-jenkins")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo '=== Pushing Petclinic Docker Image ==='
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                    SHORT_COMMIT = "${GIT_COMMIT_HASH[0..7]}"
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                        app.push("$SHORT_COMMIT")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Remove local images') {
            steps {
                echo '=== Delete the local docker images ==='
                sh("docker rmi -f 0327201/petclinic-spinnaker-jenkins:latest || :")
                sh("docker rmi -f 0327201/petclinic-spinnaker-jenkins:$SHORT_COMMIT || :")
            }
        }
    }
}
