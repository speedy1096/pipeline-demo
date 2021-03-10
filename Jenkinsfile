pipeline{
    agent any
    environment {
        DOCKER_IMAGE_NAME = "speedy1096/project-one"
        MAVEN_IMAGE_NAME = "log-aggregation-demo:0.0.1-SNAPSHOT"
    }
    stages{
        stage('Build'){
            steps{
                sh 'chmod +x mvnw && ./mvnw spring-boot:build-image'
                sh 'docker tag $MAVEN_IMAGE_NAME $DOCKER_IMAGE_NAME'
                script {
                    app = docker.image(DOCKER_IMAGE_NAME)
                }
            }
        }

        stage('Sonar Quality Analysis'){
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonarcloud'){
                    sh 'mvn -B verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar'
            }
            }
        }

        stage('Wait for Quality Gate'){
            steps{
                timeout(time: 30, unit: 'MINUTES'){
                    def qualitygate = waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Push Docker Image'){
            // when {
            //     branch 'master'
            // }
            steps{
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-jenkins-token'){
                        app.push('latest')
                        app.push("${env.BUILD_NUMBER}")
                        app.push("${env.GIT_COMMIT}")
                    }
                }
            }
        }
    }

    // post{
    //     always{

    //     }
    // }
}