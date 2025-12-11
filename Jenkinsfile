pipeline {
    agent any

    environment {
        NEXUS_URL = "http://nexus:8081"
        NEXUS_REPO = "maven-snapshots"
        NEXUS_GROUP = "com/example"
        ARTIFACT_ID = "anagrams"
    }

    stages {

        stage("BuildCode") {
            steps {
                sh """
                    cd food_order
                    ant jar
                """
            }
        }

        stage("Run-Unit-Tests") {
            steps {
                sh """
                    cd food_order
                    ant test
                """
            }
        }

        stage("Sonar-Scanning") {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                      cd food_order
                      ${tool 'SonarScanner'}/bin/sonar-scanner \
                      -Dsonar.projectKey=Food-Order-System \
                      -Dsonar.projectName=Food-Order-System \
                      -Dsonar.sources=. \
                      -Dsonar.sourceEncoding=UTF-8
                    """
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

        stage("Upload-Artifacts-Nexus") {
            steps {
                script {
                    def snapshotVersion = "1.0.0-${env.BUILD_NUMBER}-SNAPSHOT"
                    def jarPath = "${env.WORKSPACE}/food_order/dist/anagrams.jar"

                    echo "Uploading SNAPSHOT version: ${snapshotVersion}"
                    echo "Jar file path: ${jarPath}"

                    if (!fileExists(jarPath)) {
                        error "JAR file not found at ${jarPath}"
                    }

                    // Upload path to Nexus  
                    def nexusUploadPath = "${NEXUS_URL}/repository/${NEXUS_REPO}/${NEXUS_GROUP}/${ARTIFACT_ID}/${snapshotVersion}"

                    echo "Uploading to Nexus URL: ${nexusUploadPath}"

                    sh """
                        curl -v -u admin:Janu@psycho12 \
                        --upload-file ${jarPath} \
                        ${nexusUploadPath}/${ARTIFACT_ID}-${snapshotVersion}.jar
                    """

                    env.JAR_FILE_PATH = "${jarPath}"
                }
            }
        }

        stage("Deploy-Dev") {
            steps {
                sshagent(['jenkins-aws-ssh-creds']) {
                    sh """
                        scp -o StrictHostKeyChecking=no ${env.JAR_FILE_PATH} ubuntu@65.0.251.218:/home/ubuntu
                    """
                }
            }
        }

        stage("Deploy-UAT") {
            steps {
                sshagent(['jenkins-aws-ssh-creds']) {
                    sh """
                        scp -o StrictHostKeyChecking=no ${env.JAR_FILE_PATH} ubuntu@43.204.211.49:/home/ubuntu
                    """
                }
            }
        }

        stage("Deploy-PROD") {
            steps {
                sshagent(['jenkins-aws-ssh-creds']) {
                    sh """
                        scp -o StrictHostKeyChecking=no ${env.JAR_FILE_PATH} ubuntu@13.200.235.238:/home/ubuntu
                    """
                }
            }
        }
    }
}
