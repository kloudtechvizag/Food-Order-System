pipeline {
    agent any 
    stages{

        stage("BuildCode"){
            steps {
                sh """
                    cd food_order
                    ant jar
                """
            }
        }

        stage("Run-Unit-Tests"){
            steps {
                sh """
                    cd food_order
                    ant test
                """
            }
        }

        stage("Sonar-Scanning"){
            steps {
               
               script {
                    withSonarQubeEnv( installationName: 'dev-sonar-01', credentialsId: 'jenkins-sonar-creds') {
                        sh """
                            cd food_order
                            sonar-scanner \
                                        -Dsonar.projectKey=Food-Order-System \
                                        -Dsonar.projectName=Food-Order-System \
                                        -Dsonar.sources=. \
                                        -Dsonar.sourceEncoding=UTF-8
                        """
                    }
               }
            }
        }

        stage("Sonar Quality Gate") {
            steps {
                script {
                    timeout(time: 3, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline failed due to quality gate: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage("Upload-Artifacts-Nexus"){
            steps {

                script {
                    def snapshotVersion = "1.0.0-${env.BUILD_NUMBER}-SNAPSHOT"
                    def jarFileName = "${env.WORKSPACE}/food_order/dist/anagrams.jar"
                    echo "Uploading version: ${snapshotVersion}"

                    echo "Printing file name: ${jarFileName}"

                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: 'nexus:8081',
                        groupId: 'com.example',
                        version: snapshotVersion,
                        repository: 'maven-snapshots',
                        credentialsId: 'nexus-creds',
                        artifacts: [
                            [
                                artifactId: 'anagrams',
                                file: jarFileName,
                                type: "jar",
                                classifier: ""
                            ]
                        ]
                    )

                }

            }
        }

        stage("Deploy-Dev"){
            steps {
                sshagent(['jenkins-aws-ssh-creds']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no ${jarFileName} ubuntu@65.0.251.218:/home/ubuntu
                    '''
                }
            }
        }

        stage("Deploy-UAT"){
            steps {
                sshagent(['jenkins-aws-ssh-creds']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no ${jarFileName} ubuntu@43.204.211.49:/home/ubuntu
                    '''
                }
            }
        }

        stage("Deploy-PROD"){
            steps {
                sshagent(['jenkins-aws-ssh-creds']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no ${jarFileName} ubuntu@13.200.235.238:/home/ubuntu
                    '''
                }
            }
        }





    }
}