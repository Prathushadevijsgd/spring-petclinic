pipeline {
    agent any
    environment {
        DOCKER_REPO_MR = "prathushadevijs/mr"
        DOCKER_REPO_MAIN = "prathushadevijs/main"
        DOCKER_CREDENTIALS = credentials('docker-credentials-id')
    }
    
    stages {
        stage('Checkstyle') {
            agent any
            steps {
                script {
                    // Run Checkstyle with Maven (or Gradle)
                    sh 'mvn checkstyle:checkstyle'
                }
            }
            post {
                always {
                    // Archive Checkstyle Report as an artifact
                    archiveArtifacts artifacts: '**/target/checkstyle-result.xml', allowEmptyArchive: true
                }
            }
        }
        stage('Test') {
            agent any
            steps {
                script {
                    // Run Tests with Maven (or Gradle)
                    sh 'mvn test'
                }
            }
        }
        stage('Build') {
           agent any
            steps {
                script {
                    // Build the application without running tests
                    sh 'mvn clean install -DskipTests'
                }
            }
        }
        
        stage('Create Docker Image for Main Branch') {
           agent any
            when {
                // Check if the branch is 'main'
                expression {
                    return env.GIT_BRANCH ==~ /origin\/main/
                }
            }
            steps {
                script {
                    // Debugging: Print the current branch name to verify it is the correct branch
                    echo "Current Branch: ${env.GIT_BRANCH}"
                    
                    // Build Docker image for the main branch
                    if (env.GIT_BRANCH ==~ /origin\/main/) {
                        // Build Docker image for main branch
                        sh "docker build -t ${DOCKER_REPO_MAIN}/spring-petclinic:${GIT_COMMIT} ."
                        // Push Docker image to Nexus Main repository
                        sh "docker push ${DOCKER_REPO_MAIN}/spring-petclinic:${GIT_COMMIT}"
                    } else {
                        echo "Not on the main branch, skipping Docker image creation."
                    }
                }
            }
        }
    }
    post {
        failure {
            // Notify on failure
            echo 'Pipeline failed!'
        }
    }
}
