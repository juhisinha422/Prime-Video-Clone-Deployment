pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage("Clean Workspace") {
            steps { cleanWs() }
        }

        stage("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/juhisinha422/Prime-Video-Clone-Deployment.git'
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=amazon-prime \
                        -Dsonar.projectKey=amazon-prime'''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }

        stage("Install NPM Dependencies") {
            steps { sh "npm install" }
        }

        stage("OWASP Dependency Check") {
            steps {
                sh '''
                    rm -rf ~/.jenkins/dependency-check-data/odc.mv.db || true
                    rm -rf ~/.jenkins/dependency-check-data/odc.trace.db || true
                '''
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Trivy File Scan") {
            steps { sh "trivy fs . > trivy.txt" }
        }

        stage("Build Docker Image") {
            steps { sh "docker build -t amazon-prime ." }
        }

        stage("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker tag amazon-prime juhisinha/amazon-prime:latest"
                        sh "docker push juhisinha/amazon-prime:latest"
                    }
                }
            }
        }

        stage("Docker Scout Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh 'docker-scout quickview juhisinha/amazon-prime:latest'
                        sh 'docker-scout cves juhisinha/amazon-prime:latest'
                        sh 'docker-scout recommendations juhisinha/amazon-prime:latest'
                    }
                }
            }
        }

        stage("Deploy to Container") {
            steps {
                sh '''
                    docker rm -f amazon-prime || true
                    docker run -d --name amazon-prime -p 3000:3000 juhisinha/amazon-prime:latest
                '''
            }
        }
    }

    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                    <html>
                    <body>
                        <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                        </div>
                        <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                        </div>
                        <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                            <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                        </div>
                    </body>
                    </html>
                """,
                to: 'put_email_here',
                mimeType: 'text/html',
                attachmentsPattern: 'trivy.txt'
        }
    }
}
