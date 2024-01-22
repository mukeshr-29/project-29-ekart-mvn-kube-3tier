pipeline{
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/mukeshr-29/project-29-ekart-mvn-kube-3tier.git'
            }
        }
        stage('compile src code'){
            steps{
                sh 'mvn compile'
            }
        }
        stage('unit testing'){
            steps{
                sh 'mvn test -DskipTests=true'
            }
        }
        stage('static code analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar-server'){
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=ekart \
                            -Dsonar.projectName-ekart\
                            -Dsonar.java.binaries=.
                        '''
                    }
                }
            }
        }
        stage('quality gate analysis'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('owasp dependency check'){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('build application'){
            steps{
                sh 'mvn package -DskipTests=true'
            }
        }
        stage('deploy artifact to nexus repo'){
            steps{
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true){
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        stage('docker build and tag'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                        sh 'docker build -t mukeshr29/projectekart -f docker/Dockerfile .'
                    }
                }
            }
        }
        stage('trivy img scan'){
            steps{
                sh 'trivy image mukeshr29/projectekart > trivyimg.txt'
            }
        }
        stage('docker push img to dockerhub'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                        sh 'docker push mukeshr29/projectekart'
                    }
                }
            }
        }
        stage('kubernetes deployment'){
            steps{
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://10.0.18.194:6443'){
                    sh 'kubectl apply -f deploymentservice.yml -n webapps'
                    sh 'kubectl get svc -n webapps'

                }
            }
        }
    }
}