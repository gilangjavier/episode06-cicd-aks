def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    environment {
        //once you create ACR in Azure cloud, use that here
        registryName = "acrgilangjavier"
        //- update your credentials ID after creating credentials for connecting to ACR
        registryCredential = 'ACR'
        dockerImage = ''
        registryUrl = 'acrgilangjavier.azurecr.io'
    }

    stages {
        stage('Fetch code') {
            steps {
                git branch: 'main', url: 'https://github.com/gilangjavier/java-docs-spring-hello-world.git'

            }

        }
        stage('Login Azure Service Principal') {
            steps {
                sh 'az login --service-principal -u $MY_CRED_CLIENT_ID -p $MY_CRED_CLIENT_SECRET -t $MY_CRED_TENANT_ID'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn install -DskipTest'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=java-docs-spring-hello-world \
                    -Dsonar.projectName=java-docs-spring-hello-world \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=src/test/java/com/example/demo/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \

                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Upload Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '20.169.104.139:8081',
                    groupId: 'com.example',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'javarest-repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'java-docs-spring-hello-world',
                         classifier: '',
                         file: 'target/java-docs-spring-hello-world-0.0.1-SNAPSHOT.jar',
                         type: 'jar']
                    ]
                )
            }
        }
        stage('Deploy to Azure App Services') {
            steps {
                azureWebAppPublish azureCredentialsId: 'AzurePrincipal',
                   resourceGroup: 'TEST', appName: 'gillvision',
                   filePath: '*.war', sourceDirectory: '**/*.jar', targetDirectory: 'webapps'
            }
            post {
                success {
                    echo 'Deploy WebApp Success'
                }
             }
         }
    }
}